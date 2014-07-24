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
    "name": "Retopology Tools",
    "author": "Cédric Lepiller", "Gert De Roost for the Laprelax code" 
    "version": (0, 1, 0),
    "blender": (2, 7, 0),
    "location": "View 3D > Toolbar > Tools tab > Retopology (panel)",
    "description": "Tools for fast retopology",
    "category": "3D View"}

############### Operators

import bpy
import bmesh
from mathutils import *
import math

#Double Threshold 0.001
class DoubleThreshold0001(bpy.types.Operator):
    bl_idname = "double.threshold0001"
    bl_label = "Double Threshold 0001"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    def execute(self, context):
        bpy.context.scene.tool_settings.double_threshold = 0.001
        return {'FINISHED'}

#Double Threshold 0.1
class DoubleThreshold01(bpy.types.Operator):
    bl_idname = "double.threshold01"
    bl_label = "Double Threshold 01"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    def execute(self, context):
        bpy.context.scene.tool_settings.double_threshold = 0.1
        return {'FINISHED'}

#LapRelax
class LapRelax(bpy.types.Operator):
	bl_idname = "mesh.laprelax"
	bl_label = "LapRelax"
	bl_description = "Smoothing mesh keeping volume"
	bl_options = {'REGISTER', 'UNDO'}
	
	Repeat = bpy.props.IntProperty(
		name = "Repeat", 
		description = "Repeat how many times",
		default = 1,
		min = 1,
		max = 100)


	@classmethod
	def poll(cls, context):
		obj = context.active_object
		return (obj and obj.type == 'MESH' and context.mode == 'EDIT_MESH')

	def invoke(self, context, event):
		
		# smooth #Repeat times
		for i in range(self.Repeat):
			self.do_laprelax()
		
		return {'FINISHED'}


	def do_laprelax(self):
	
		context = bpy.context
		region = context.region  
		area = context.area
		selobj = bpy.context.active_object
		mesh = selobj.data
		bm = bmesh.from_edit_mesh(mesh)
		bmprev = bm.copy()
	
		for v in bmprev.verts:
			if v.select:
				tot = Vector((0, 0, 0))
				cnt = 0
				for e in v.link_edges:
					for f in e.link_faces:
						if not(f.select):
							cnt = 1
					if len(e.link_faces) == 1:
						cnt = 1
						break
				if cnt:
					# dont affect border edges: they cause shrinkage
					continue
					
				# find Laplacian mean
				for e in v.link_edges:
					tot += e.other_vert(v).co
				tot /= len(v.link_edges)
				
				# cancel movement in direction of vertex normal
				delta = (tot - v.co)
				if delta.length != 0:
					ang = delta.angle(v.normal)
					deltanor = math.cos(ang) * delta.length
					nor = v.normal
					nor.length = abs(deltanor)
					bm.verts[v.index].co = tot + nor
			
			
		mesh.update()
		bm.free()
		bmprev.free()
		bpy.ops.object.editmode_toggle()
		bpy.ops.object.editmode_toggle()

#Wire on all objects
class Wire_All(bpy.types.Operator):
    """Tooltip"""
    bl_idname = "object.wire_all"
    bl_label = "Wire on All Objects"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None

    def execute(self, context):
        
        for obj in bpy.data.objects:
            if obj.show_wire:
                obj.show_all_edges = False
                obj.show_wire = False
            
            else:
                obj.show_all_edges = True
                obj.show_wire = True
               
                
        return {'FINISHED'}

#Align to X
class AlignToX(bpy.types.Operator):  
    bl_idname = "object.align2x"  
    bl_label = "Align To X"  
  
    def execute(self, context):
        bpy.ops.object.mode_set(mode = 'OBJECT')
        bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)

        for vert in bpy.context.object.data.vertices:
            if vert.select: 
                vert.co[0] = 0
        bpy.ops.object.editmode_toggle() 
        return {'FINISHED'} 
"""
#Select Ngons
class SelectNGons1(bpy.types.Operator):  
    bl_idname = "object.sngons1"  
    bl_label = "Select Ngons"  
  
    def execute(self, context):
        print("----- SCRIPT -----")
            
        for ob in bpy.data.objects:
            if ob.select:
                if ob.type == "MESH":
                    ob.select = True
                    m = ob.data
                    faces = []
                    for face in m.polygons:
                        face.select = False
                        verts_on_face = face.vertices[:] 
                        if len(verts_on_face) > 4:
                            face.select = True
                            faces.append(face.index)
                                              
                    if len(faces) > 0:
                        ob.select = False
                        print(ob.name)                
                        print(faces)
                        bpy.context.scene.objects.active = ob                  
                        
            bpy.ops.object.mode_set(mode = 'EDIT')
            return {'FINISHED'} 
"""
#################### Panel
  
class RetopologyTools(bpy.types.Panel): 
    
    bl_label = "Retopology Tools"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'TOOLS'
    bl_category = "Retopology"
  
    def draw(self, context): 
        layout = self.layout 
        
        #Tools
        layout.label(text="Tools :")
        
        row = layout.row()
        row.operator("mesh.laprelax")
        row = layout.row()
        row.operator("object.align2x", icon='MOD_WIREFRAME')
        row = layout.row()
        #row.operator("object.sngons1", icon='MESH_ICOSPHERE')
        split = layout.split()
        split = layout.split()
        row = layout.row()
        row.operator("mesh.flip_normals", icon = 'FULLSCREEN_ENTER')
        row = layout.row()
        row.operator("mesh.normals_make_consistent", icon = 'MATCUBE')
        row = layout.row()
        row.operator("mesh.remove_doubles",icon='X_VEC')
        row = layout.row()
        #row.operator("object.join", icon = 'ROTATECENTER')
        #row.operator("mesh.separate", icon = 'ROTATECOLLECTION').type = "LOOSE"
        row = layout.row()
        #Shading
        layout.label(text="Modifiers :")
        row = layout.row()
        row.operator("object.modifier_apply", text="Apply Subsurf", icon='MOD_SUBSURF').modifier="Subsurf"
        row.operator("object.modifier_apply", text="Apply Mirror", icon='MOD_MIRROR').modifier="Mirror"
        
        row = layout.row()
        row.operator("object.modifier_remove", text="Del Subsurf", icon='MOD_SUBSURF').modifier="Subsurf"
        row.operator("object.modifier_remove", text="Del Mirror", icon='MOD_MIRROR').modifier="Mirror"

        #Shading
        layout.label(text="Shading :")
        row = layout.row()
        row.operator("wm.context_toggle", text="Xray", icon='META_CUBE').data_path = "object.show_x_ray"
        row.operator("wm.context_toggle", text="HIWR", icon='GHOST_ENABLED').data_path = "space_data.show_occlude_wire"
        row.operator("wm.context_toggle", text="L2V", icon='ORTHO').data_path = "space_data.use_occlude_geometry"
        row = layout.row()
        
        row.operator("object.wire_all", text="Wire All", icon='WIRE')
        row.operator("object.wire_selected", text="Wire Selected", icon='WIRE')
        row = layout.row()
        row.operator("mesh.faces_shade_smooth", text= "Smooth", icon = 'ANTIALIASED')
        row.operator("mesh.faces_shade_flat", text= "Flat", icon = 'ALIASED')
        
        row = layout.row()
        view = context.space_data
        obj = context.object
        col = layout.column()
        col.prop(view, "show_backface_culling")
        
        row.operator("wm.context_toggle", text="D Sided", icon='MESH_DATA').data_path = "object.data.show_double_sided"
        row.operator("wm.context_toggle", text="A Smooth", icon='MESH_DATA').data_path = "object.data.use_auto_smooth"

        
        #Properties
        layout.label(text="Properties :")
        ob = context.active_object

        tool_settings = context.tool_settings
        mesh = ob.data

        col = layout.column(align=True)
        col.prop(mesh, "use_mirror_x")
        
        row = layout.row()
        row.operator("double.threshold01", text= "TSHD 0.1")
        row.operator("double.threshold0001", text= "TSHD 0.001")
        
        row = layout.row()
        col.label("Double Threshold:")
        col.prop(tool_settings, "double_threshold", text="")
        col = layout.column(align=True)
        
        row = layout.row()
        row.operator("wm.context_toggle", text="Auto Merge", icon='AUTOMERGE_ON').data_path = "scene.tool_settings.use_mesh_automerge"
        
        
  
def register(): 
    bpy.utils.register_class(RetopologyTools) 
    #bpy.utils.register_class(SelectNGons1)
    bpy.utils.register_class(AlignToX)
    bpy.utils.register_class(Wire_All)
    bpy.utils.register_class(LapRelax)
    bpy.utils.register_class(DoubleThreshold01)
    bpy.utils.register_class(DoubleThreshold0001)
    
    
    
    
def unregister(): 
    bpy.utils.unregister_class(RetopologyTools) 
    #bpy.utils.unregister_class(SelectNGons1)
    bpy.utils.unregister_class(AlignToX)
    bpy.utils.unregister_class(Wire_All)
    bpy.utils.unregister_class(LapRelax)
    bpy.utils.unregister_class(DoubleThreshold01)
    bpy.utils.unregister_class(DoubleThreshold0001)
    
    
if __name__ == "__main__": 
    register() 







