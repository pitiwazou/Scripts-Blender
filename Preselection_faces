# ##### BEGIN GPL LICENSE BLOCK #####
#
#  3dview_edit_preselect.py
#  Draw the mesh element under the cursor thicker in Edit Mode
#  Copyright (C) 2015 Quentin Wenger
#
#  This program is free software; you can redistribute it and/or
#  modify it under the terms of the GNU General Public License
#  as published by the Free Software Foundation; either version 2
#  of the License, or (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program; if not, write to the Free Software Foundation,
#  Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.
#
# ##### END GPL LICENSE BLOCK #####



bl_info = {"name": "Edit Preselect",
           "description": "Draw the mesh element under the cursor " \
                          "thicker in Edit Mode.",
           "author": "Quentin Wenger (Matpi)",
           "version": (1, 0),
           "blender": (2, 75, 0),
           "location": "3D View(s) -> Properties -> Shading",
           "warning": "",
           "wiki_url": "",
           "tracker_url": "",
           "category": "3D View"
           }


import bpy
#from bpy_extras.view3d_utils import location_3d_to_region_2d
from bpy_extras.view3d_utils import (region_2d_to_origin_3d,
                                     region_2d_to_vector_3d)
from mathutils import Vector
from mathutils.geometry import intersect_ray_tri, tessellate_polygon
from bmesh import from_edit_mesh
from bgl import (glColor4f, glEnable, glDisable, glBegin, glEnd,
                 glVertex3f, glLineWidth, glPointSize)
from bgl import (GL_BLEND, GL_DEPTH_TEST, GL_POLYGON, GL_LINES, GL_POINTS)

RAY_MAX = 100.0

# maximum distance triggering an action
# should be consistent with Blender...
DIST_MAX = 27
DIST_MAX_SQR = DIST_MAX**2

class ViewPreselectWorker(object):

#    ACTIVE = 0
#    SELECT = 1
#    NORMAL = 2

    def __init__(self):

        self.handle = None
        self.mouse_x = 0
        self.mouse_y = 0
        self.region = None
        self.region_3d = None
        self.ray_origin = Vector((0.0, 0.0, 0.0))
        self.ray_direction = Vector((0.0, 0.0, 0.0))
        self.bmesh = None
        self.select_verts = False
        self.select_edges = False
        self.select_faces = False
        self.running = False
        self.flag = False
        self.flag_redraw = True

        self.draw_face = None
        self.draw_edges = []
        self.draw_verts = []

#        self.use_custom_color = False
        self.custom_color = (0.0, 1.0, 0.0)
        self.face_opacity = 0.0
        self.edge_opacity = 1.0
        self.vert_opacity = 1.0
        self.depth_test = False
        self.edge_width = 3.0
        self.vert_radius = 3.0

    def set_glColor_vert(self):
        glColor4f(self.custom_color[0],
                  self.custom_color[1],
                  self.custom_color[2],
                  self.vert_opacity)

    def set_glColor_edge(self):
        glColor4f(self.custom_color[0],
                  self.custom_color[1],
                  self.custom_color[2],
                  self.edge_opacity)

    def set_glColor_face(self):
        glColor4f(self.custom_color[0],
                  self.custom_color[1],
                  self.custom_color[2],
                  self.face_opacity)
    
    def updateSelectionModes(self, context):
        (self.select_verts,
         self.select_edges,
         self.select_faces) = context.tool_settings.mesh_select_mode

    def drawCallback(self):
        if self.running and bpy.context.mode == 'EDIT_MESH' and not self.flag:

            glEnable(GL_BLEND)
            if self.depth_test:
                glEnable(GL_DEPTH_TEST)
            else:
                glDisable(GL_DEPTH_TEST)

            if self.select_faces and self.face_opacity and self.draw_face is not None:
                glBegin(GL_POLYGON)

                self.set_glColor_face()

                for vertex in self.draw_face:
                    glVertex3f(*vertex)
                
                glEnd()

            if ((self.select_edges or self.select_faces)
                and self.edge_opacity and self.draw_edges):
                glLineWidth(self.edge_width)
                
                glBegin(GL_LINES)

                self.set_glColor_edge()

                for edge in self.draw_edges:
                    glVertex3f(*edge[0])
                    glVertex3f(*edge[1])
                
                glEnd()

            if self.select_verts and self.vert_opacity and self.draw_verts:
                glPointSize(self.vert_radius)
                
                glBegin(GL_POINTS)

                self.set_glColor_vert()

                for vertex in self.draw_verts:
                    glVertex3f(*vertex)
                
                glEnd()

            glLineWidth(1.0)
            glPointSize(1.0)

            # if multiple redraws without modal running inbetween
            # -> face could have been changed/... (i.e. other modal running)
            # so avoid drawing outdated data
            self.flag = True

    def location_3d_to_region_2d(self, coord):
        """
        Modified copy of bpy_extras.view3d_utils.location_3d_to_region_2d
        without negative w coord check
        """
        from mathutils import Vector

        prj = self.region_3d.perspective_matrix*Vector((coord[0],
                                                        coord[1],
                                                        coord[2],
                                                        1.0))
        width_half = self.region.width/2.0
        height_half = self.region.height/2.0
        
        if prj.w == 0.0:
            # hope it never happens...
            prj.w = -0.0001

        return (width_half*(1.0 + prj.x/prj.w),
                height_half*(1.0 + prj.y/prj.w))
    
    def testEdge2D(self, coords):
        x0, y0 = coords[0]
        x1, y1 = coords[1]

        xa = x1 - x0
        ya = y1 - y0

        x = self.mouse_x - x0
        y = self.mouse_y - y0

        dist_sqr = xa*xa + ya*ya

        if dist_sqr == 0.0:
            return None

        s = (x*xa + y*ya)/dist_sqr

        if s < 0.0:
            return x*x + y*y

        elif s > 1.0:
            return (x - xa)**2 + (y - ya)**2
            
        else:
            return ((xa*y - ya*x)**2)/dist_sqr

    def testVert2D(self, coords):
        x = self.mouse_x - coords[0]
        y = self.mouse_y - coords[1]

        return x*x + y*y
    
    def testTri2D(self, coords):
        x0, y0 = coords[0]
        x1, y1 = coords[1]
        x2, y2 = coords[2]

        xa = x1 - x0
        ya = y1 - y0

        xb = x2 - x0
        yb = y2 - y0

        x = self.mouse_x - x0
        y = self.mouse_y - y0

        det = xa*yb - ya*xb

        if det == 0.0:
            return False

        s = (x*yb - y*xb)/det
        t = (xa*y - ya*x)/det
        
        return s >= 0.0 and t >= 0.0 and s + t <= 1.0

    def testTri3D(self, coords):
        inter = intersect_ray_tri(coords[0],
                                  coords[1],
                                  coords[2],
                                  self.ray_direction,
                                  self.ray_origin)
        if inter is None:
            return None
        return (inter - self.ray_origin).length

    def modal(self, context, event):
        self.mouse_x = event.mouse_region_x
        self.mouse_y = event.mouse_region_y

        self.region = context.region

        if context.space_data is not None:
            self.region_3d = context.space_data.region_3d

            self.updateSelectionModes(context)

            # self.flag_redraw -> one more redraw needed at the end of modals
            # (which block this very one)
            if self.flag and not self.flag_redraw:
                self.region.tag_redraw()
                self.flag_redraw = True
            elif self.flag_redraw:
                self.flag_redraw = False
                
            self.flag = False

            if (self.region_3d is not None and context.mode == 'EDIT_MESH'
                and self.running):

                coords = (self.mouse_x, self.mouse_y)

                self.ray_origin = region_2d_to_origin_3d(
                    self.region,
                    self.region_3d,
                    coords)
                self.ray_direction = region_2d_to_vector_3d(
                    self.region,
                    self.region_3d,
                    coords)*RAY_MAX

                obj = context.object
                matrix_world = obj.matrix_world

                if self.bmesh is None or not self.bmesh.is_valid:
                    self.bmesh = from_edit_mesh(obj.data)

                df = self.draw_face
                de = self.draw_edges
                dv = self.draw_verts

                found = False

                if self.select_verts:

                    min_dist_sqr = None
                    point_vert = None

                    for vert in self.bmesh.verts:
                        if not vert.hide:
                            point = matrix_world*vert.co
                            point_2d = self.location_3d_to_region_2d(point)

                            dist_sqr = self.testVert2D(point_2d)

                            if dist_sqr is not None:
                                if (min_dist_sqr is None
                                    or min_dist_sqr > dist_sqr):
                                    min_dist_sqr = dist_sqr
                                    point_vert = point

                    if point_vert is None or min_dist_sqr > DIST_MAX_SQR:
                        #????
                        self.draw_face = None
                        self.draw_edges = []
                        self.draw_verts = []

                    else:
                        found = True
                        self.draw_face = None
                        self.draw_edges = []
                        self.draw_verts = [point_vert]


                if self.select_edges and not found:

                    min_dist_sqr = None
                    points_edge = None

                    for edge in self.bmesh.edges:
                        if not edge.hide:
                            points = [matrix_world*v.co for v in edge.verts]
                            points_2d = [self.location_3d_to_region_2d(point)
                                         for point in points]

                            dist_sqr = self.testEdge2D(points_2d)

                            if dist_sqr is not None:
                                if (min_dist_sqr is None
                                    or min_dist_sqr > dist_sqr):
                                    min_dist_sqr = dist_sqr
                                    points_edge = points

                    if points_edge is None or min_dist_sqr > DIST_MAX_SQR:
                        #????
                        self.draw_face = None
                        self.draw_edges = []
                        self.draw_verts = []

                    else:
                        found = True
                        self.draw_face = None
                        self.draw_edges = [points_edge]
                        self.draw_verts = points_edge
                        

                if self.select_faces and not found:

                    min_dist = None
                    points_face = None
                    
                    for face in self.bmesh.faces:
                        if not face.hide:
                            points = [matrix_world*v.co for v in face.verts]
                            points_2d = [self.location_3d_to_region_2d(point)
                                         for point in points]
                            tess_face = tessellate_polygon((points,))

                            possible_tris = []

                            for tri in tess_face:
                                if self.testTri2D([points_2d[i] for i in tri]):
                                    possible_tris.append(tri)

                            for tri in possible_tris:
                                dist = self.testTri3D([points[i] for i in tri])
                                if dist is not None:
                                    if min_dist is None or min_dist > dist:
                                        min_dist = dist
                                        points_face = points
                                        break

                    if points_face is None:
                        self.draw_face = None
                        self.draw_edges = [] # XXX TEMP!!!
                        self.draw_verts = [] # XXX
                    else:
                        self.draw_face = points_face
                        self.draw_edges = [(points_face[i - 1], v)
                                           for i, v in enumerate(points_face)]
                        self.draw_verts = points_face

                if (df != self.draw_face or de != self.draw_edges
                    or dv != self.draw_verts):
                    self.region.tag_redraw()
                    # forced redraw -> not another modal
                    # -> no need for a new one
                    self.flag_redraw = True


    def register(self):
        if self.handle is not None:
            bpy.types.SpaceView3D.draw_handler_remove(self.handle, 'WINDOW')
        self.handle = bpy.types.SpaceView3D.draw_handler_add(
            self.drawCallback,
            (),
            'WINDOW',
            'POST_VIEW')

    def unregister(self):
        if self.handle is not None:
            bpy.types.SpaceView3D.draw_handler_remove(self.handle, 'WINDOW')
            self.handle = None




class ViewOperatorPreselect(bpy.types.Operator):
    bl_idname = "view3d.modal_operator_preselect"
    bl_label = "Preselect View Operator"
    bl_options = {'INTERNAL'}

    worker = ViewPreselectWorker()

    def modal(self, context, event):
        self.worker.modal(context, event)
        return {'PASS_THROUGH'}

    def invoke(self, context, event):
        if context.space_data.type == 'VIEW_3D':
            context.window_manager.modal_handler_add(self)
            return {'RUNNING_MODAL'}
        else:
            # should not happen
            self.report({'WARNING'}, "Active space must be a View3d")
            return {'CANCELLED'}

    @classmethod
    def register(cls):
        cls.worker.register()
    
    @classmethod
    def unregister(cls):
        cls.worker.unregister()


def updateBGLData(self, context):
    VOP = ViewOperatorPreselect
    
    if self.preselect_use:
        if not VOP.worker.running:
            # INVOKE_DEFAULT leads to error in cursor position retrieval
            # (position relative to properties shelf rather than 3d view)
            bpy.ops.view3d.modal_operator_preselect('INVOKE_REGION_WIN')
            VOP.worker.running = True
        VOP.worker.edge_width = self.preselect_width
        VOP.worker.vert_radius = self.preselect_radius
#        VOP.worker.use_custom_color = self.custom_color_use
        VOP.worker.custom_color = self.custom_color
        VOP.worker.face_opacity = self.opacity_face
        VOP.worker.edge_opacity = self.opacity_edges
        VOP.worker.vert_opacity = self.opacity_vertices
        VOP.worker.depth_test = self.depth_test
        return

    VOP.worker.running = False


class PreselectCollectionGroup(bpy.types.PropertyGroup):
    preselect_use = bpy.props.BoolProperty(
        name="Preselect Faces",
        description="Display edges of the face under the cursor thicker",
        default=False,
        update=updateBGLData)
    depth_test = bpy.props.BoolProperty(
        name="Depth Test",
        description="Redraw elements only if they are closer (zbuff test)",
        default=False,
        update=updateBGLData)
    preselect_width = bpy.props.FloatProperty(
        name="Width",
        description="Edges width in pixels",
        min=1.0,
        max=10.0,
        default=3.0,
        subtype='PIXEL',
        update=updateBGLData)
    preselect_radius = bpy.props.FloatProperty(
        name="Radius",
        description="Vertices radius in pixels",
        min=1.0,
        max=10.0,
        default=3.0,
        subtype='PIXEL',
        update=updateBGLData)
    opacity_vertices = bpy.props.FloatProperty(
        name="Vertices Opacity",
        description="Opacity of the preselection color for vertices",
        min=0.0,
        max=1.0,
        default=1.0,
        subtype='FACTOR',
        update=updateBGLData)
    opacity_edges = bpy.props.FloatProperty(
        name="Edges Opacity",
        description="Opacity of the preselection color for edges",
        min=0.0,
        max=1.0,
        default=1.0,
        subtype='FACTOR',
        update=updateBGLData)
    opacity_face = bpy.props.FloatProperty(
        name="Face Opacity",
        description="Opacity of the preselection color for the face",
        min=0.0,
        max=1.0,
        default=0.0,
        subtype='FACTOR',
        update=updateBGLData)
    custom_color = bpy.props.FloatVectorProperty(
        name="Custom Color",
        description="Color to draw elements with",
        min=0.0,
        max=1.0,
        default=(0.0, 1.0, 0.0),
        size=3,
        subtype='COLOR',
        update=updateBGLData)
#    custom_color_use = bpy.props.BoolProperty(
#        name="Custom Color",
#        description="Use a unique color to draw edges with",
#        default=False,
#        update=updateBGLData)


def displayPreselectPanel(self, context):
    layout = self.layout

    preselect = context.window_manager.preselect

    if context.mode == 'EDIT_MESH':
        layout.prop(preselect, "preselect_use")

        if preselect.preselect_use:

            v, e, f = context.tool_settings.mesh_select_mode
            
            split = layout.split(percentage=0.1)
            split.separator()

            col = split.column()
            
            col.prop(preselect, "depth_test")

            r = col.row()
            r.active = v
            r.prop(preselect, "preselect_radius")
            r = col.row()
            r.active = e or f
            r.prop(preselect, "preselect_width")
            
            r = col.row()
            r.active = v
            r.prop(preselect, "opacity_vertices")
            r = col.row()
            r.active = e or f
            r.prop(preselect, "opacity_edges")
            r = col.row()
            r.active = f
            r.prop(preselect, "opacity_face")

#            if preselect.custom_color_use:
#                split2 = col.split()    
#                split2.prop(preselect, "custom_color_use")
            col.prop(preselect, "custom_color", text="")
#            else:
#                col.prop(preselect, "custom_color_use") 


def register():
    bpy.utils.register_module(__name__)
    bpy.types.WindowManager.preselect = bpy.props.PointerProperty(
        type=PreselectCollectionGroup)
    bpy.types.VIEW3D_PT_view3d_shading.append(displayPreselectPanel)
    

def unregister():
    bpy.types.VIEW3D_PT_view3d_shading.remove(displayPreselectPanel)
    del bpy.types.WindowManager.preselect
    bpy.utils.unregister_module(__name__)
    

if __name__ == "__main__":
    register()
