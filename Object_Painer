# ##### BEGIN GPL LICENSE BLOCK #####
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

bl_info = {
    "name": "Object Painter",
    "category": "Object",
    "author": "Blenderlounge",
    "version": (1,0),
    "blender": (2, 71, 0),
    "location": "3DView->Object Painter Panel (T-Key)",
    "description": "Allows to place objects on a canvas.",
}


import bpy
import mathutils
import random
from math import radians, degrees, cos, sin, sqrt
import bpy_extras
from bpy_extras import view3d_utils
import bgl
import blf
#import IntProperty, FloatProperty, BoolProperty

########################
#      Properties      #               
########################

class ObjectPaintPrefs(bpy.types.AddonPreferences):
    """Creates the tools in a Panel, in the scene context of the properties editor"""
    bl_idname = __name__

    bpy.types.Scene.Enable_Tab_OP_01 = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.Enable_Tab_OP_02 = bpy.props.BoolProperty(default=False)
    
    def draw(self, context):
        layout = self.layout
        
        layout.prop(context.scene, "Enable_Tab_OP_01", text="Authors", icon="QUESTION")  
        if context.scene.Enable_Tab_OP_01:
            row = layout.row()
            layout.label(text="– Author > Will Souloumiac")
            layout.label(text="– Author > Cédric Lepiller")
            layout.label(text="– Author > Blendelounge")
            
        layout.prop(context.scene, "Enable_Tab_OP_02", text="URL's", icon="URL") 
        if context.scene.Enable_Tab_OP_02:
            row = layout.row()    
            
            row.operator("wm.url_open", text="BlenderLounge Forum ").url = "http://blenderlounge.fr/forum/"   
            row.operator("wm.url_open", text="Pitiwazou.com").url = "http://www.pitiwazou.com/"
            row.operator("wm.url_open", text="Wazou's Ghitub").url = "https://github.com/pitiwazou/Scripts-Blender"
            

#########################################################################################################


#########################################################################################################
def draw_callback_px(self, context):
    if(self.ShowCursor):
        region = bpy.context.region
        rv3d = bpy.context.space_data.region_3d
        view_width = context.region.width
        bgl.glEnable(bgl.GL_BLEND)
        bgl.glLineWidth(2)
        bgl.glColor4f(1.0, 0.8, 0.0, 1.0)
        bgl.glBegin(bgl.GL_LINE_STRIP)
        idx = 0
        for i in range(int(len(self.CLR_C)/3)):
            vector3d = (self.CLR_C[idx * 3] * self.CRadius + self.CurLoc.x,self.CLR_C[idx * 3  + 1] * self.CRadius + self.CurLoc.y,self.CLR_C[idx * 3 + 2] * self.CRadius + self.CurLoc.z)
            vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(region, rv3d, vector3d)
            bgl.glVertex2f(*vector2d)
            idx += 1
        bgl.glEnd()
        bgl.glBegin(bgl.GL_LINE_STRIP)
        idx = 0
        for i in range(int(len(self.CLR_C)/3)):
            vector3d = (self.CLR_C[idx * 3] * self.CRadius/3.0 + self.CurLoc.x,self.CLR_C[idx * 3  + 1] * self.CRadius/3.0 + self.CurLoc.y,self.CLR_C[idx * 3 + 2] * self.CRadius/3.0 + self.CurLoc.z)
            vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(region, rv3d, vector3d)
            bgl.glVertex2f(*vector2d)
            idx += 1
        bgl.glEnd()
        bgl.glLineWidth(1)
        bgl.glDisable(bgl.GL_BLEND)
        bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
#########################################################################################################


#########################################################################################################
def CreatePrimitive(self, _AngleStep, _radius):
    CLRaw = []
    Angle = 0.0
    self.NbPointsInPrimitive = 0
    while(Angle < 360.0):
        self.CircleListRaw.append(cos(radians(Angle)) * _radius)
        self.CircleListRaw.append(sin(radians(Angle)) * _radius)
        self.CircleListRaw.append(0.0)
        Angle += _AngleStep
        self.NbPointsInPrimitive += 1
    self.CircleListRaw.append(cos(radians(0.0)) * _radius)
    self.CircleListRaw.append(sin(radians(0.0)) * _radius)
    self.CircleListRaw.append(0.0)
    self.NbPointsInPrimitive += 1

#########################################################################################################


################################################################################################
def RBenVe(Object, Dir):
    ObjectV = Object.normalized()
    DirV = Dir.normalized()
    cosTheta = ObjectV.dot(DirV)
    rotationAxis = mathutils.Vector((0.0, 0.0, 0.0))
    if (cosTheta < -1 + 0.001):
        v = mathutils.Vector((0.0, 1.0, 0.0))
        rotationAxis = ObjectV.cross(v)
        rotationAxis = rotationAxis.normalized()
        q = mathutils.Quaternion()
        q.w = 0.0
        q.x = rotationAxis.x
        q.y = rotationAxis.y
        q.z = rotationAxis.z
        return q
    rotationAxis = ObjectV.cross(DirV);
    s = sqrt( (1.0+cosTheta)*2.0 );
    invs = 1 / s;
    q = mathutils.Quaternion()
    q.w = s * 0.5
    q.x = rotationAxis.x * invs
    q.y = rotationAxis.y * invs
    q.z = rotationAxis.z * invs
    return q
################################################################################################


################################################################################################
def duplicateObject(scene, copyobj, qRot, location, self):
    mesh = bpy.data.meshes.new(copyobj.name)
    ob_new = bpy.data.objects.new(copyobj.name, mesh)
    if(self.Instanciate):
        ob_new.data = copyobj.data
    else:        
        ob_new.data = copyobj.data.copy()
    ob_new.location = location

    if(copyobj.Uniform_OPT == True):
        if(copyobj.ScaleXmin_OPT != copyobj.ScaleXmax_OPT):
            ob_new.scale.x = float(random.randrange(int(copyobj.ScaleXmin_OPT * 100), int(copyobj.ScaleXmax_OPT * 100))/100.0)
            ob_new.scale.y = ob_new.scale.z = ob_new.scale.x
        else:
            ob_new.scale.y = ob_new.scale.z = ob_new.scale.x = 1.0
    else:            
        if(copyobj.ScaleXmin_OPT != copyobj.ScaleXmax_OPT):
            ob_new.scale.x = float(random.randrange(int(copyobj.ScaleXmin_OPT * 100), int(copyobj.ScaleXmax_OPT * 100))/100.0)
        else:
             ob_new.scale.x = 1.0       
        if(copyobj.ScaleYmin_OPT != copyobj.ScaleYmax_OPT):
            ob_new.scale.y = float(random.randrange(int(copyobj.ScaleYmin_OPT * 100), int(copyobj.ScaleYmax_OPT * 100))/100.0)
        else:
             ob_new.scale.y = 1.0       
        if(copyobj.ScaleZmin_OPT != copyobj.ScaleZmax_OPT):
            ob_new.scale.z = float(random.randrange(int(copyobj.ScaleZmin_OPT * 100), int(copyobj.ScaleZmax_OPT * 100))/100.0)
        else:
             ob_new.scale.z = 1.0       
    e = mathutils.Euler()
    e.x = e.y = 0.0
    if(copyobj.RotXmin_OPT != copyobj.RotXmax_OPT):
        e.x = radians(float(random.randrange(int(copyobj.RotXmin_OPT * 100.0), int(copyobj.RotXmax_OPT * 100.0)))/100.0)
    else:
        e.x = 0.0
    if(copyobj.RotYmin_OPT != copyobj.RotYmax_OPT):
        e.y = radians(float(random.randrange(int(copyobj.RotYmin_OPT * 100.0), int(copyobj.RotYmax_OPT * 100.0)))/100.0)
    else:
        e.y = 0.0
    if(copyobj.RotZmin_OPT != copyobj.RotZmax_OPT):
        e.z = radians(float(random.randrange(int(copyobj.RotZmin_OPT * 100.0), int(copyobj.RotZmax_OPT * 100.0)))/100.0)
    else:
        e.z = 0.0
    qe = e.to_quaternion()
    qRot = qRot * qe
    ob_new.rotation_mode = 'QUATERNION'
    ob_new.rotation_quaternion = qRot
    ob_new.rotation_mode = 'XYZ'
    scene.objects.link(ob_new)
    ob_new.select = False
    
    for emp in self.emptiesName:
        if(emp[0] == copyobj.name):
            ob_new.select = False
            bpy.context.scene.objects.active = bpy.data.objects[emp[1]]
            bpy.data.objects[emp[1]].hide = False
            bpy.data.objects[emp[1]].select = True
            ob_new.select = True
            bpy.ops.object.parent_set(type = "OBJECT", keep_transform = False)                                
            ob_new.select = False
            bpy.data.objects[emp[1]].select = False
            bpy.data.objects[emp[1]].hide = True

            break
    
    return ob_new
################################################################################################

################################################################################################
def MoveCursor(qRot, location, self):
    if(qRot != None):
        self.CLR_C.clear()
        vc = mathutils.Vector()        
        idx = 0
        for i in range(int(len(self.CircleListRaw)/3)):
            vc.x = self.CircleListRaw[idx * 3] * self.CRadius
            vc.y = self.CircleListRaw[idx * 3  + 1] * self.CRadius
            vc.z = self.CircleListRaw[idx * 3 + 2] * self.CRadius
            vc = qRot * vc
            self.CLR_C.append(vc.x)
            self.CLR_C.append(vc.y)
            self.CLR_C.append(vc.z)
            idx += 1
################################################################################################

################################################################################################
def Pick(context, event, self, ray_max=10000.0):
    scene = context.scene
    region = context.region
    rv3d = context.region_data
    coord = event.mouse_region_x, event.mouse_region_y
    view_vector = view3d_utils.region_2d_to_vector_3d(region, rv3d, coord)
    ray_origin = view3d_utils.region_2d_to_origin_3d(region, rv3d, coord)
    ray_target = ray_origin + (view_vector * ray_max)

    def obj_ray_cast(obj, matrix):
        matrix_inv = matrix.inverted()
        ray_origin_obj = matrix_inv * ray_origin
        ray_target_obj = matrix_inv * ray_target
        hit, normal, face_index = obj.ray_cast(ray_origin_obj, ray_target_obj)
        if face_index != -1:
            return hit, normal, face_index
        else:
            return None, None, None

    best_length_squared = ray_max * ray_max
    best_obj = None
    for obj in self.CList:
        matrix = obj.matrix_world
        hit, normal, face_index = obj_ray_cast(obj, matrix)
        if hit is not None:
            hit_world = matrix * hit
            length_squared = (hit_world - ray_origin).length_squared
            if length_squared < best_length_squared:
                best_length_squared = length_squared
                best_obj = obj
                hits = hit_world
                ns = normal
                fs = face_index
            
    if best_obj is not None:
        return hits, ns, fs
    else:
        return None, None, None
################################################################################################

################################################################################################
def PickObject(context, event, self, ray_max=10000.0):
    scene = context.scene
    region = context.region
    rv3d = context.region_data
    coord = event.mouse_region_x, event.mouse_region_y
    view_vector = view3d_utils.region_2d_to_vector_3d(region, rv3d, coord)
    ray_origin = view3d_utils.region_2d_to_origin_3d(region, rv3d, coord)
    ray_target = ray_origin + (view_vector * ray_max)

    def obj_ray_cast(obj, matrix):
        matrix_inv = matrix.inverted()
        ray_origin_obj = matrix_inv * ray_origin
        ray_target_obj = matrix_inv * ray_target
        hit, normal, face_index = obj.ray_cast(ray_origin_obj, ray_target_obj)
        if face_index != -1:
            return hit, normal, face_index
        else:
            return None, None, None

    for obj in bpy.context.visible_objects:
        if(obj.type == "MESH"):
            fcanv = False
            for oc in self.CList:
                if(oc == obj):
                    fcanv = True
            for oc in self.OPList:
                if(oc == obj):
                    fcanv = True
            if(fcanv == False):
                matrix = obj.matrix_world
                hit, normal, face_index = obj_ray_cast(obj, matrix)
                if hit is not None:
                    hit_world = matrix * hit
                    return obj
                
    return None
################################################################################################


################################################################################################
class ObjectPaintModal(bpy.types.Operator):
    bl_idname = "view3d.modal_objectpaint"
    bl_label = "Object Paint Operator"

    def modal(self, context, event):
        context.area.tag_redraw()
        if event.type in {'MIDDLEMOUSE', 'WHEELUPMOUSE', 'WHEELDOWNMOUSE'}:
            if(self.anchored):
                if(self.LeftM == False):
                    return {'PASS_THROUGH'}
            else:                
                return {'PASS_THROUGH'}
        elif event.type == 'MOUSEMOVE':
            vBack = Pick(context, event, self)
            if(vBack[0] != None):
                self.ShowCursor = True   
                if(self.anchored and (self.Cut == False)):
                    if(self.LeftM == False):
                        NormalObject = mathutils.Vector((0.0, 0.0, 1.0))
                        qR = RBenVe(NormalObject, vBack[1])
                        MoveCursor(qR, vBack[0], self)
                        self.CurLoc = vBack[0]
                else:                        
                    NormalObject = mathutils.Vector((0.0, 0.0, 1.0))
                    qR = RBenVe(NormalObject, vBack[1])
                    MoveCursor(qR, vBack[0], self)
                    self.CurLoc = vBack[0]
            else:
                self.ShowCursor = False                    

            if(self.anchored == False):
                if((self.LeftM == True) and (vBack[0] != None)):
                    if(self.Cut == False):
                        if(self.LastPck == None):
                            self.LastPck = vBack[0]
                        lgth = vBack[0] - self.LastPck
                        if(lgth.length_squared > self.MinDist):
                            if(self.Placed == False):
                                vBack = Pick(context, event, self)
                                if(vBack[0] != None):
                                    NormalObject = mathutils.Vector((0.0, 0.0, 1.0))
                                    qR = RBenVe(NormalObject, vBack[1])
                                    no = duplicateObject(bpy.context.scene, self.OPList[self.RList[random.randrange(0, len(self.RList) - 1)]], qR, vBack[0], self)
                                    item = bpy.context.scene.Undo_OPT.add()
                                    item.ObjectName = no.name
                                    
                                    self.Placed = True
                                self.LastPck = vBack[0]
                            else:
                                self.Placed = False
                            self.LastPosition = event.mouse_region_x, event.mouse_region_y 
                    else:                        
                        bpy.context.window.cursor_modal_set("KNIFE")
                        vBack = PickObject(context, event, self)
                        if(vBack != None):
                            vBack.select = True
                            bpy.ops.object.delete()
            else:
                if(self.LeftM):
                    if(self.Cut == False):
                        self.aRotZ = event.mouse_region_x - self.am[0]
                        self.ascale  = event.mouse_region_y - self.am[1]
                        self.am = event.mouse_region_x, event.mouse_region_y
                        if(self.aObj != None):
                            if(self.CRadius >= 0.0):
                                self.CRadius += float(self.ascale)/50.0
                                if(self.CRadius < 0.0):
                                    self.CRadius = 0.0
                                
                            self.aObj.scale.x += float(self.ascale)/50.0
                            if(self.aObj.scale.x <= 0.0):
                                self.aObj.scale.x = 0.0
                            self.aObj.scale.y += float(self.ascale)/50.0
                            if(self.aObj.scale.y <= 0.0):
                                self.aObj.scale.y = 0.0
                            self.aObj.scale.z += float(self.ascale)/50.0
                            if(self.aObj.scale.z <= 0.0):
                                self.aObj.scale.z = 0.0
                            e = mathutils.Euler()
                            e.x = e.y = 0.0
                            e.z = self.aRotZ/25.0
                            qe = e.to_quaternion()
                            self.aqR = self.aqR * qe
                            self.aObj.rotation_mode = 'QUATERNION'
                            self.aObj.rotation_quaternion = self.aqR
                            self.aObj.rotation_mode = 'XYZ'
                    else:
                        bpy.context.window.cursor_modal_set("KNIFE")
                        vBack = PickObject(context, event, self)
                        if(vBack != None):
                            vBack.select = True
                            bpy.ops.object.delete()
        elif event.type == 'LEFTMOUSE':
            if event.value == 'PRESS':
                self.LeftM = True
                if(event.ctrl):
                    self.Cut = True
                if(self.LeftM == True):
                    if(self.Cut == False):
                        if ((pow((self.LastPosition[0] - event.mouse_region_x), 2) + pow((self.LastPosition[1] - event.mouse_region_y), 2)) > self.MinDist):                
                            if(self.Placed == False):
                                vBack = Pick(context, event, self)
                                if(vBack[0] != None):
                                    NormalObject = mathutils.Vector((0.0, 0.0, 1.0))
                                    self.aqR = RBenVe(NormalObject, vBack[1])
                                    self.aObj = duplicateObject(bpy.context.scene, self.OPList[self.RList[random.randrange(0, len(self.RList) - 1)]], self.aqR, vBack[0], self)
                                    self.am = event.mouse_region_x, event.mouse_region_y
                                    item = bpy.context.scene.Undo_OPT.add()
                                    item.ObjectName = self.aObj.name
                                    self.Placed = True
                            else:
                                self.Placed = False
                            self.LastPosition = event.mouse_region_x, event.mouse_region_y 

                        self.LastPosition = event.mouse_region_x, event.mouse_region_y 
                    else:
                        bpy.context.window.cursor_modal_set("KNIFE")
                        vBack = PickObject(context, event, self)
                        if(vBack != None):
                            for o in bpy.context.selected_objects:
                                o.select = False
                            vBack.select = True
                            bpy.ops.object.delete()
            "if(event.value == 'RELEASE'):"
            if(event.shift):
                if(event.value == 'PRESS'):
                    self.anchored = True
                else:                    
                    self.anchored = False
                                    
            if(event.value == 'RELEASE'):
                self.Cut = False
                self.LeftM = False 
                self.Placed = False       
                self.CRadius = 1.0
                self.LastPck = None
                bpy.context.window.cursor_modal_set("DEFAULT")
        elif event.type in {'RIGHTMOUSE', 'ESC'}:
            bpy.types.SpaceView3D.draw_handler_remove(self._handle, 'WINDOW')
            bpy.data.scenes[0].RunObjectPaint = False
            return {'CANCELLED'}
        return {'RUNNING_MODAL'}

    def invoke(self, context, event):
        if (context.space_data.type == 'VIEW_3D'):
            self.Placed = False
            self.LeftM = False
            self.LastPosition = event.mouse_region_x, event.mouse_region_y 
            self.MinDist = bpy.data.scenes[0].distance
            self.Instanciate = bpy.data.scenes[0].instance
            bpy.data.scenes[0].RunObjectPaint = True
            self.NbO = 0
            self.Cut = False
            self.anchored = bpy.data.scenes[0].anchored
            self.aObj = None
            self.am = -1, -1
            self.ascale = 0
            self.aRotZ = 0
            self.aqR = None
            self.CList = []
            self.OPList = []
            self.RList = []
            for ent in bpy.data.scenes[0].Canevas:
                obj = bpy.data.objects[ent.description]
                if(obj.Checked_C):
                    self.CList.append(obj)

            iO = 0
            for ent in bpy.data.scenes[0].ObjectPaint:
                obj = bpy.data.objects[ent.description]
                if(obj.Checked_OPT):
                    self.NbO += 1
                    self.OPList.append(obj)
                    for i in range(int(obj.Ratio_OPT/10)):
                        self.RList.append(iO)
                    iO += 1                        

            args = (self, context)
            self._handle = bpy.types.SpaceView3D.draw_handler_add(draw_callback_px, args, 'WINDOW', 'POST_PIXEL')
            
            bpy.context.scene.Undo_OPT.clear()
            
            self.LastPck = None
            
            self.CircleListRaw = []
            self.CLR_C = []
            self.CurLoc = mathutils.Vector((0.0, 0.0, 0.0))
            self.CRadius = 1.0
            CreatePrimitive(self, 10.0, 1.0)
            self.VertsList = []
            self.FacesList = []
            
            self.ShowCursor = True
            
            for o in bpy.context.selected_objects:
                o.select = False
            # Creation des empties
            bACreated = False
            scn = bpy.context.scene
            for o in scn.objects:
                if(o.name == "OP Group"):
                    bACreated = True
            if(bACreated == False):
                bpy.ops.object.empty_add()
                bpy.context.selected_objects[0].name = "OP Group"
                bpy.data.objects["OP Group"].select = False
                
            # Creation des empties pour les object paint
            self.emptiesName = []
            for e in bpy.data.scenes[0].ObjectPaint:
                name = e.description
                bACreated = False
                for o in scn.objects:
                    if(o.name == "OP " + name):
                        bACreated = True
                        break
                if(bACreated == False):
                    bpy.ops.object.empty_add()
                    bpy.context.selected_objects[0].name = "OP " + name
                    bpy.data.objects["OP " + name].select = False
                    bpy.ops.object.empty_add()
                    bpy.ops.object.delete()

                    bpy.context.scene.objects.active = bpy.data.objects["OP Group"]
                    bpy.data.objects["OP Group"].hide = False
                    bpy.data.objects["OP Group"].select = True
                    bpy.data.objects["OP " + name].select = True
                    bpy.ops.object.parent_set(type = "OBJECT", keep_transform = False)                                
                    self.emptiesName.append((name, "OP " + name))
                    bpy.data.objects["OP Group"].select = False
                    bpy.data.objects["OP " + name].select = False
                    bpy.data.objects["OP " + name].hide = True
                    bpy.data.objects["OP Group"].hide = True
                else:
                    self.emptiesName.append((name, "OP " + name))
    

            context.window_manager.modal_handler_add(self)
            return {'RUNNING_MODAL'}
        else:
            bpy.data.scenes[0].RunObjectPaint = False
            self.report({'WARNING'}, "Active space must be a View3d")
            return {'CANCELLED'}
################################################################################################



################################################################################################
class CanevasEntry(bpy.types.PropertyGroup):
    label = bpy.props.FloatProperty()
    description = bpy.props.StringProperty()
    idx = bpy.props.IntProperty()


class CANEVAS_UL_search(bpy.types.UIList):
    def draw_item(self, context, layout, data, item, icon, active_data, active_propname):
        if self.layout_type in {'DEFAULT', 'COMPACT'}:
            split = layout.split(percentage=0.75, align = True)
            split.label(item.description)
            split.label(" ")
            split.prop(bpy.data.objects[item.description], "Checked_C", text = "")
            split.operator("object.canevasop", text="", icon="PANEL_CLOSE").type = ("remove" + str(item.idx))
            split.operator("object.canevasop", text="", icon="RESTRICT_SELECT_OFF").type = ("select" + str(item.idx))
        elif self.layout_type in {'GRID'}:
            pass

class CanevasOp(bpy.types.Operator):
    bl_idname = "object.canevasop"
    bl_label = "Canevas Operator"
    bl_description = "Objects canvas"

    type=bpy.props.StringProperty(default="")
    
    def execute(self, context):
        type=self.type
        if(type=="add"):
            scn = context.scene
            for nobj in context.selected_objects:
                if(nobj.type == 'MESH'):
                    item = scn.Canevas.add()
                    item.description = nobj.name
                    item.idx = len(bpy.data.scenes[0].Canevas) - 1
                    object = nobj
                    object.Checked_OPT = True
                    object.select = False
            bpy.data.scenes[0].Canevas_idx = len(bpy.data.scenes[0].Canevas) - 1
                    
        if(type[:6]=="select"):
            scn = context.scene
            bpy.data.objects[scn.Canevas[int(type[6:])].description].select = True
        if(type=="removeAll"):
            scn = context.scene
            scn.Canevas.clear()
            bpy.data.scenes[0].Canevas_idx = -1
        elif(type[:6]=="remove"):
            scn = context.scene
            scn.Canevas.remove(int(type[6:]))
            if(bpy.data.scenes[0].Canevas_idx >= len(bpy.data.scenes[0].Canevas)):
                bpy.data.scenes[0].Canevas_idx = len(bpy.data.scenes[0].Canevas) - 1
            for i in range(int(type[6:]), len(bpy.data.scenes[0].Canevas)):
                scn.Canevas[i].idx -= 1
                
            
            
        return {'FINISHED'}
################################################################################################


################################################################################################
class UndoEntry(bpy.types.PropertyGroup):
    ObjectName = bpy.props.StringProperty()

class UndoOp(bpy.types.Operator):
    bl_idname = "object.undoop"
    bl_label = "UndoOPT Operator"
    bl_description = "Objects undo OPT"

    type=bpy.props.StringProperty(default="")
    
    def execute(self, context):
        type=self.type
        
        if(len(bpy.context.scene.Undo_OPT) > 0):
            for o in bpy.context.selected_objects:
                o.select = False
            for o in bpy.context.scene.Undo_OPT:
                try:
                    bpy.data.objects[o.ObjectName].select = True
                except:
                    print("")
            bpy.ops.object.delete()
        return {'FINISHED'}
################################################################################################



################################################################################################
class ObjectPaintEntry(bpy.types.PropertyGroup):
    description = bpy.props.StringProperty()
    icon = bpy.props.StringProperty(default="OBJECT_DATA")
    idx = bpy.props.IntProperty()

class OBJPAINT_UL_search(bpy.types.UIList):
    def draw_item(self, context, layout, data, item, icon, active_data, active_propname):
        if self.layout_type in {'DEFAULT', 'COMPACT'}:
            split = layout.split(percentage=0.75, align=True)
            split.prop(bpy.data.objects[item.description], "Ratio_OPT", text=item.description, slider=True)       
            split.label(" ")
            split.prop(bpy.data.objects[item.description], "Checked_OPT", text = "")
            split.operator("object.objpaintop", text="", icon="PANEL_CLOSE").type = ("remove" + str(item.idx))
            split.operator("object.objpaintop", text="", icon="RESTRICT_SELECT_OFF").type = ("select" + str(item.idx))
        elif self.layout_type in {'GRID'}:
            pass

class ObjectPaintOp(bpy.types.Operator):
    bl_idname = "object.objpaintop"
    bl_label = "Object Paint Operator"
    bl_description = "Objects paint"

    type=bpy.props.StringProperty(default="")
    
    def execute(self, context):
        type=self.type
        if(type=="add"):
            scn = context.scene
            for nobj in context.selected_objects:
                if(nobj.type == "MESH"):
                    item = scn.ObjectPaint.add()
                    item.purcent = 100
                    item.description = nobj.name
                    item.idx = len(bpy.data.scenes[0].ObjectPaint) - 1
                    object = nobj
                    nobj.ScaleXmin_OPT = 1.0
                    nobj.ScaleXmax_OPT = 1.0
                    nobj.ScaleYmin_OPT = 1.0
                    nobj.ScaleYmax_OPT = 1.0
                    nobj.ScaleZmin_OPT = 1.0
                    nobj.ScaleZmax_OPT = 1.0
                    nobj.RotXmin_OPT = 0.0
                    nobj.RotXmax_OPT = 0.0
                    nobj.RotYmin_OPT = 0.0
                    nobj.RotYmax_OPT = 0.0
                    nobj.RotZmin_OPT = 0.0
                    nobj.RotZmax_OPT = 0.0
                    nobj.Checked_OPT = True
                    nobj.Ratio_OPT = 100
                    nobj.select = False
            bpy.data.scenes[0].ObjectPaint_idx = len(bpy.data.scenes[0].ObjectPaint) - 1
            
        if(type[:6]=="select"):
            scn = context.scene
            name = bpy.data.objects[scn.ObjectPaint[int(type[6:])].description].name
            if(name.rfind('.') >= 0):
                name = name[:(name.rfind('.'))]
            print(name)
            if(len(bpy.context.scene.Undo_OPT) > 0):
                for o in bpy.context.selected_objects:
                    o.select = False
                for o in bpy.context.scene.Undo_OPT:
                    try:
                        oname = bpy.data.objects[o.ObjectName].name
                        oname = oname[:(oname.rfind('.'))]
                        if(oname == name):
                            bpy.data.objects[o.ObjectName].select = True
                    except:
                        print("ObjectPainter : Error in selection")                            
        if(type=="removeAll"):
            scn = context.scene
            scn.ObjectPaint.clear()
            bpy.data.scenes[0].ObjectPaint_idx = -1
        elif (type[:6]=="remove"):
            scn = context.scene
            scn.ObjectPaint.remove(int(type[6:]))
            if(bpy.data.scenes[0].ObjectPaint_idx >= len(bpy.data.scenes[0].ObjectPaint)):
                bpy.data.scenes[0].ObjectPaint_idx = len(bpy.data.scenes[0].ObjectPaint) - 1
            for i in range(int(type[6:]), len(bpy.data.scenes[0].ObjectPaint)):
                scn.ObjectPaint[i].idx -= 1

        
        return {'FINISHED'}
################################################################################################


################################################################################################
class ObjectPainter(bpy.types.Panel):
    bl_label = "Object Painter"
    bl_idname = "OBJECT_PT_painter"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'TOOLS'
    bl_context = "objectmode"
    bl_category = "Object Painter"

    def draw(self, context):
        layout = self.layout
        scene = context.scene
        box = layout.box()
        box = box.split(percentage=0.8, align=True)
        if(bpy.data.scenes[0].RunObjectPaint == True):
            box.operator("view3d.modal_objectpaint", text = "Disable ESC or Right click", icon ="TPAINT_HLT")
        else:
            box.operator("view3d.modal_objectpaint", text = "Paint Objects", icon ="TPAINT_HLT")
        box.prop(scene, "anchored", text="", icon = "SNAP_NORMAL")
        
        nopC = nopOP = False
        for o in bpy.data.scenes[0].objects:
            for e in bpy.data.scenes[0].Canevas:
                if(e.description == o.name):
                    nopC = True
                    break
            if(nopC):
                break
        for o in bpy.data.scenes[0].objects:
            for e in bpy.data.scenes[0].ObjectPaint:
                if(e.description == o.name):
                    nopOP = True
                    break
            if(nopOP):
                break
            
        if(nopC and nopOP):
            if((len(bpy.data.scenes[0].ObjectPaint) > 0) and (len(bpy.data.scenes[0].Canevas) > 0) and (bpy.data.scenes[0].RunObjectPaint == False)):
                bObjPaint = False
                bCanevas = False
                
                try:
                    for e in bpy.data.scenes[0].ObjectPaint:
                        if((bpy.data.objects[e.description].Checked_OPT) == True):
                            bObjPaint = True
                            break
                except:
                    print("ObjectPainter : 'Object paint' not found")                    
                try:
                    for e in bpy.data.scenes[0].Canevas:
                        if((bpy.data.objects[e.description].Checked_C) == True):
                            bCanevas = True
                            break
                except:
                    print("ObjectPainter : 'Canevas' not found")                    

                if(bObjPaint and bCanevas):                
                    box.active = True
                else:
                    box.active = False
            else:
                box.active = False
        else:
            box.active = False
           
        row = box.row()
        box.operator("object.undoop", text = "", icon = "LOOP_BACK").type = "removeAll"  
        layout.label("Press Shift to be Enchored mode")  
        layout.label("Press Ctrl to Delete objects")
            
        split = layout.split(percentage = 0.95, align=True)
        split.operator("object.canevasop", text = "Add canvas", icon ="ZOOMIN").type = "add"
        split.operator("object.canevasop", text = "", icon ="PANEL_CLOSE").type = "removeAll"
        layout.template_list("CANEVAS_UL_search", "", scene, "Canevas", scene, "Canevas_idx", rows = 1)
        
        layout.separator()
        row = layout.split(percentage=0.5,align=True)
        row.operator("object.objpaintop", text="Add object to paint", icon = "MOD_DYNAMICPAINT").type = "add"
        row = row.split(percentage = 0.15, align = True)
        row.operator("object.objpaintop", text = "", icon = "PANEL_CLOSE").type = "removeAll"
        row.prop(scene, "instance", text="Instanciate")        
        layout.template_list("OBJPAINT_UL_search", "", scene, "ObjectPaint", scene, "ObjectPaint_idx", rows = 1)

        if(bpy.data.scenes[0].anchored == False):
            if(bpy.data.scenes[0].ObjectPaint_idx >= 0):
                entry = bpy.data.scenes[0].ObjectPaint[bpy.data.scenes[0].ObjectPaint_idx]
                layout.separator()
                split = layout.split(percentage=0.7)
                split.label("Scale", icon = 'MAN_SCALE')
                split.prop(bpy.data.objects[entry.description], "Uniform_OPT", text="Uniform scale")
                box = layout.box()
                row = box.row(align = True)
                if(bpy.data.objects[entry.description].Uniform_OPT):
                    split = row.split(percentage=0.2, align = True)
                    split.label("Scale")
                    split.prop(bpy.data.objects[entry.description], "ScaleXmin_OPT", text="min", slider = True)
                    split.prop(bpy.data.objects[entry.description], "ScaleXmax_OPT", text="max", slider = True)
                else:
                    split = row.split(percentage=0.2, align = True)
                    split.label("Scale X")
                    split.prop(bpy.data.objects[entry.description], "ScaleXmin_OPT", text="min", slider = True)
                    split.prop(bpy.data.objects[entry.description], "ScaleXmax_OPT", text="max", slider = True)
                    row = box.row(align = True)
                    split = row.split(percentage=0.2, align = True)
                    split.label("Scale Y")
                    split.prop(bpy.data.objects[entry.description], "ScaleYmin_OPT", text="min", slider = True)
                    split.prop(bpy.data.objects[entry.description], "ScaleYmax_OPT", text="max", slider = True)
                    row = box.row(align = True)
                    split = row.split(percentage=0.2, align = True)
                    split.label("Scale Z")
                    split.prop(bpy.data.objects[entry.description], "ScaleZmin_OPT", text="min", slider = True)
                    split.prop(bpy.data.objects[entry.description], "ScaleZmax_OPT", text="max", slider = True)
                       
                layout.separator()
                layout.label("Rotation", icon = 'MAN_ROT')
                box = layout.box()
                row = box.row(align = True)
                split = row.split(percentage=0.2, align = True)
                split.label("X axis")
                split.prop(bpy.data.objects[entry.description], "RotXmin_OPT", text="min", slider=True)       
                split.prop(bpy.data.objects[entry.description], "RotXmax_OPT", text="max", slider=True)       
                row = box.row(align = True)
                split = row.split(percentage=0.2, align = True)
                split.label("Y axis")
                split.prop(bpy.data.objects[entry.description], "RotYmin_OPT", text="min", slider=True)       
                split.prop(bpy.data.objects[entry.description], "RotYmax_OPT", text="max", slider=True)       
                row = box.row(align = True)
                split = row.split(percentage=0.2, align = True)
                split.label("Z axis")
                split.prop(bpy.data.objects[entry.description], "RotZmin_OPT", text="min", slider=True)       
                split.prop(bpy.data.objects[entry.description], "RotZmax_OPT", text="max", slider=True)       

            layout.separator()
            layout.label("Distance between objects", icon = 'ARROW_LEFTRIGHT')
            box = layout.box()
            box.prop(scene, "distance", text="Distance")        

        
            

def register():
    bpy.utils.register_module(__name__)
    
    bpy.types.Scene.Canevas = bpy.props.CollectionProperty(type=CanevasEntry)
    bpy.types.Scene.Canevas_idx = bpy.props.IntProperty(default=0)
    bpy.types.Scene.ObjectPaint = bpy.props.CollectionProperty(type=ObjectPaintEntry)
    bpy.types.Scene.ObjectPaint_idx = bpy.props.IntProperty(default=0)
    bpy.types.Scene.Undo_OPT = bpy.props.CollectionProperty(type=UndoEntry)
    
    bpy.types.Object.ScaleXmin_OPT = bpy.props.FloatProperty(name="ScaleXmin_OPT",default=1.0, min = 0.001, max = 50.0)
    bpy.types.Object.ScaleYmin_OPT = bpy.props.FloatProperty(name="ScaleYmin_OPT",default=1.0, min = 0.001, max = 50.0)
    bpy.types.Object.ScaleZmin_OPT = bpy.props.FloatProperty(name="ScaleZmin_OPT",default=1.0, min = 0.001, max = 50.0)
    bpy.types.Object.Uniform_OPT = bpy.props.BoolProperty(name="Uniform_OPT",default=True)

    bpy.types.Object.ScaleXmax_OPT = bpy.props.FloatProperty(name="ScaleXmax_OPT",default=1.0, min = 0.001, max = 50.0)
    bpy.types.Object.ScaleYmax_OPT = bpy.props.FloatProperty(name="ScaleYmax_OPT",default=1.0, min = 0.001, max = 50.0)
    bpy.types.Object.ScaleZmax_OPT = bpy.props.FloatProperty(name="ScaleZmax_OPT",default=1.0, min = 0.001, max = 50.0)

    bpy.types.Object.RotXmin_OPT = bpy.props.FloatProperty(name="RotXmin_OPT",default=0.0, min = -180.0, max = 0.0)
    bpy.types.Object.RotYmin_OPT = bpy.props.FloatProperty(name="RotYmin_OPT",default=0.0, min = -180.0, max = 0.0)
    bpy.types.Object.RotZmin_OPT = bpy.props.FloatProperty(name="RotZmin_OPT",default=0.0, min = -180.0, max = 0.0)

    bpy.types.Object.RotXmax_OPT = bpy.props.FloatProperty(name="RotXmax_OPT",default=0.0, min = 0.0, max = 180.0)
    bpy.types.Object.RotYmax_OPT = bpy.props.FloatProperty(name="RotYmax_OPT",default=0.0, min = 0.0, max = 180.0)
    bpy.types.Object.RotZmax_OPT = bpy.props.FloatProperty(name="RotZmax_OPT",default=0.0, min = 0.0, max = 180.0)

    bpy.types.Object.Checked_OPT = bpy.props.BoolProperty(name="Checked_OPT",default=True)
    bpy.types.Object.Checked_C = bpy.props.BoolProperty(name="Checked_C",default=True)
    bpy.types.Object.Ratio_OPT = bpy.props.IntProperty(name="Ratio_OPT",default=100, min=0, max = 100)


    bpy.types.Scene.distance = bpy.props.FloatProperty(default=2.0, min = 0.0, max = 100.0)
    bpy.types.Scene.instance = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.anchored = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.RunObjectPaint = bpy.props.BoolProperty(default=False)


def unregister():
    bpy.utils.unregister_module(__name__)
    
    del bpy.types.Scene.Canevas
    del bpy.types.Scene.Canevas_idx
    del bpy.types.Scene.ObjectPaint
    del bpy.types.Scene.ObjectPaint_idx

    del bpy.types.Object.ScaleXmin_OPT
    del bpy.types.Object.ScaleYmin_OPT
    del bpy.types.Object.ScaleZmin_OPT
    del bpy.types.Object.Uniform_OPT

    del bpy.types.Object.ScaleXmax_OPT
    del bpy.types.Object.ScaleYmax_OPT
    del bpy.types.Object.ScaleZmax_OPT

    del bpy.types.Object.RotXmin_OPT
    del bpy.types.Object.RotYmin_OPT
    del bpy.types.Object.RotZmin_OPT

    del bpy.types.Object.RotXmax_OPT
    del bpy.types.Object.RotYmax_OPT
    del bpy.types.Object.RotZmax_OPT

    del bpy.types.Object.Checked_OPT
    del bpy.types.Object.Checked_C

    del bpy.types.Object.Ratio_OPT

    del bpy.types.Scene.distance
    del bpy.types.Scene.instance
    del bpy.types.Scene.RunObjectPaint



if __name__ == "__main__":
    register()

