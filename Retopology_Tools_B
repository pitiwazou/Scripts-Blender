# ##### BEGIN GPL LICENSE BLOCK ###############################################      
#                                                                             #
#  This program is free software; you can redistribute it and/or              #
#  modify it under the terms of the GNU General Public License                #
#  as published by the Free Software Foundation; either version 2             #
#  of the License, or (at your option) any later version.                     #
#                                                                             #
#  This program is distributed in the hope that it will be useful,            #
#  but WITHOUT ANY WARRANTY; without even the implied warranty of             #
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the              #
#  GNU General Public License for more details.                               #
#                                                                             #
#  You should have received a copy of the GNU General Public License          #
#  along with this program; if not, write to the Free Software Foundation,    #
#  Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301, USA.         #
#                                                                             #
# ##### END GPL LICENSE BLOCK #################################################

###############################################################################################################################################################################
#
# This B version of the retopology tool needs Iceking's Addon, Bsurfaces, Looptools and Auto mirror and surface Constraint tools to work
# - Iceking's Addon > http://www.blenderartists.org/forum/showthread.php?343641-Iceking-s-Tools
# - Auto Mirror By Lapineige > http://le-terrier-de-lapineige.over-blog.com/2014/07/automirror-mon-add-on-pour-symetriser-vos-objets-rapidement.html
# - Bsurface 1.5 > http://www.blenderartists.org/forum/showthread.php?225190-Bsurfaces-v1-5&highlight=bsurface
# - Surface Constraint tools > http://www.blenderartists.org/forum/showthread.php?350794-Addon-Surface-Constraint-Tools
#
###############################################################################################################################################################################



bl_info = {
    "name": "Retopology Tools",
    "author": "Cédric Lepiller", "Gert De Roost for the Laprelax code" 
    "version": (0, 1, 4),
    "blender": (2, 7, 2),
    "location": "View 3D > Toolbar > Tools tab > Retopology (panel)",
    "description": "Tools for fast retopology",
    "category": "3D View"}




import bpy, os
from bpy.types import Menu, Header  
import bmesh
from mathutils import *
import math
from bpy.props import IntProperty, FloatProperty, BoolProperty


############### Prefs ########################

#Class Prefs
class WazouRetopoTools(bpy.types.AddonPreferences):
    """Creates the tools in a Panel, in the scene context of the properties editor"""
    bl_idname = __name__

    bpy.types.Scene.Enable_Tab_01 = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.Enable_Tab_02 = bpy.props.BoolProperty(default=False)
    

    def draw(self, context):
        layout = self.layout

        layout.prop(context.scene, "Enable_Tab_01", text="Info", icon="QUESTION")
        if context.scene.Enable_Tab_01:
            row = layout.row()
            layout.label(text="This Addon Need to activate 'Loop Tools' and 'Bsurfaces' in the Addon Tab to work properly.")
            layout.label(text="You need to install Iceking's Tool, Surface Constraint tools And Auto Mirror")
            
            layout.operator("wm.url_open", text="IceKing's tools").url = "http://www.blenderartists.org/forum/showthread.php?343641-Iceking-s-Tools"
            layout.operator("wm.url_open", text="Auto Mirror").url = "http://le-terrier-de-lapineige.over-blog.com/2014/07/automirror-mon-add-on-pour-symetriser-vos-objets-rapidement.html"
            layout.operator("wm.url_open", text="Surface Constraint tools").url = "http://www.blenderartists.org/forum/showthread.php?350794-Addon-Surface-Constraint-Tools"
            
            
        layout.prop(context.scene, "Enable_Tab_02", text="URL's", icon="URL")   
        if context.scene.Enable_Tab_02:
            row = layout.row()
            row.operator("wm.url_open", text="Pitiwazou.com").url = "http://www.pitiwazou.com/"
            row.operator("wm.url_open", text="Wazou's Ghitub").url = "https://github.com/pitiwazou/Scripts-Blender"
            row.operator("wm.url_open", text="BlenderLounge Forum ").url = "http://blenderlounge.fr/forum/"

############### Operators ########################

#Setup Retopo Mesh
class SetupRetopoMesh(bpy.types.Operator):  
    bl_idname = "setup.retopomesh"  
    bl_label = "Setup Retopo Mesh" 
    bl_options = {'REGISTER', 'UNDO'} 
  
    def execute(self, context):
        bpy.ops.setup.retopo()
        bpy.context.object.show_x_ray = True
        bpy.context.space_data.show_occlude_wire = True
        return {'FINISHED'} 

#Double Threshold 0.001
class DoubleThreshold0001(bpy.types.Operator):
    bl_idname = "double.threshold0001"
    bl_label = "Double Threshold 0001"
    bl_options = {'REGISTER', 'UNDO'}

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
    bl_options = {'REGISTER', 'UNDO'}

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

#Align to X
class AlignToX(bpy.types.Operator):  
    bl_idname = "object.align2x"  
    bl_label = "Align To X"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        bpy.ops.object.mode_set(mode = 'OBJECT')
        bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)

        for vert in bpy.context.object.data.vertices:
            if vert.select: 
                vert.co[0] = 0
        bpy.ops.object.editmode_toggle() 
        return {'FINISHED'} 

#Wire on selected objects
class WireSelectedAll(bpy.types.Operator):
    """Wire on selected Object, Wire All if no objects selected"""
    bl_idname = "wire.selectedall"
    bl_label = "Wire Selected All"
    bl_options = {'REGISTER', 'UNDO'}

    @classmethod
    def poll(cls, context):
        return context.active_object is not None

    def execute(self, context):
             
        for obj in bpy.data.objects:
            if bpy.context.selected_objects:
                if obj.select:
                    if obj.show_wire:
                        obj.show_all_edges = False
                        obj.show_wire = False
                    else:
                        obj.show_all_edges = True
                        obj.show_wire = True
            elif not bpy.context.selected_objects:
                if obj.show_wire:
                    obj.show_all_edges = False
                    obj.show_wire = False
                else:
                    obj.show_all_edges = True
                    obj.show_wire = True
        return {'FINISHED'}
    
#Shading Smooth
class ShadingSmooth(bpy.types.Operator):
    bl_idname = "shading.smooth"
    bl_label = "Shading Smooth"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.shade_smooth()
            
        elif bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.shade_smooth()
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'} 

#Shading Flat    
class ShadingFlat(bpy.types.Operator):
    bl_idname = "shading.flat"
    bl_label = "Shading Flat"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.shade_flat()
            
        elif bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.shade_flat()
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'} 

#Create Hole
class CreateHole(bpy.types.Operator):                  
    """This Operator create a hole on a selection"""                   
    bl_idname = "object.createhole"                     
    bl_label = "Create Hole"   
    bl_options = {'REGISTER', 'UNDO'}     

    @classmethod                                     
    def poll(cls, context):                         
        return context.active_object is not None 

    def execute(self, context):                     
        
        bpy.ops.mesh.extrude_region_move()
        bpy.ops.transform.resize(value=(0.6, 0.6, 0.6))
        bpy.ops.mesh.looptools_circle()
        bpy.ops.mesh.extrude_region_move()
        bpy.ops.transform.resize(value=(0.8, 0.8, 0.8))
        bpy.ops.mesh.delete(type='FACE')
        return {'FINISHED'}     

#Delete modifiers
class DeleteModifiers(bpy.types.Operator):  
    bl_idname = "delete.modifiers"  
    bl_label = "Delete modifiers"
    bl_options = {'REGISTER', 'UNDO'} 
    
    def execute(self, context):
        selection = bpy.context.selected_objects
        
        if not(selection):  
            for obj in bpy.data.objects:
                for mod in obj.modifiers:
                    bpy.context.scene.objects.active = obj
                    bpy.ops.object.modifier_remove(modifier = mod.name)
        else:
            for obj in selection:
                for mod in obj.modifiers:
                    bpy.context.scene.objects.active = obj
                    bpy.ops.object.modifier_remove(modifier = mod.name)  
        return {'FINISHED'}  

#Add Mirror Object
class AddMirrorObject(bpy.types.Operator):  
    bl_idname = "add.mirrorobject"  
    bl_label = "Add Mirror Object"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.modifier_add(type='MIRROR')
            bpy.context.object.modifiers["Mirror"].use_clip = True
            bpy.context.object.modifiers["Mirror"].show_on_cage = True
            
        elif bpy.context.object.mode == "EDIT":
            bpy.ops.object.modifier_add(type='MIRROR')
            bpy.context.object.modifiers["Mirror"].use_clip = True
            bpy.context.object.modifiers["Mirror"].show_on_cage = True
            bpy.context.object.modifiers["Mirror"].show_in_editmode = True

        return {'FINISHED'}  
    
#Apply Mirror
class ApplyMirror(bpy.types.Operator):  
    bl_idname = "apply.mirror"  
    bl_label = "Apply Mirror"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.modifier_apply(apply_as='DATA', modifier="Mirror")

            
        elif bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.modifier_apply(apply_as='DATA', modifier="Mirror")
            bpy.ops.object.mode_set(mode = 'EDIT')
        
        return {'FINISHED'}   
      
#Subsurf 2
class SubSurf2(bpy.types.Operator):  
    bl_idname = "object.subsurf2"  
    bl_label = "SubSurf 2"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.subdivision_set(level=2)
            bpy.context.object.modifiers["Subsurf"].show_only_control_edges = True
        
        if bpy.context.object.mode == "EDIT":
            bpy.ops.object.subdivision_set(level=2)
            bpy.context.object.modifiers["Subsurf"].show_on_cage = True
            bpy.context.object.modifiers["Subsurf"].show_only_control_edges = True
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'}   

#Remove Subsurf
class RemoveSubSurf2(bpy.types.Operator):  
    bl_idname = "object.removesubsurf2"  
    bl_label = "Remove SubSurf"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.modifier_remove(modifier="Subsurf")
        
        if bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.modifier_remove(modifier="Subsurf")
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'}
    
#Apply Subsurf
class ApplySubSurf(bpy.types.Operator):  
    bl_idname = "object.applysubsurf"  
    bl_label = "Apply subsurf"  
    bl_options = {'REGISTER', 'UNDO'}
  
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.modifier_apply(apply_as='DATA', modifier="Subsurf")

        if bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.modifier_apply(apply_as='DATA', modifier="Subsurf")
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'}   
 
#Sursurf On/Off
class SursurfOnOff(bpy.types.Operator):
    bl_idname = "object.subsurfonoff"
    bl_label = "Sursurf On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Subsurf"].show_viewport = not bpy.context.object.modifiers["Subsurf"].show_viewport
        return {'FINISHED'} 
    
#Sursurf Edit On/Off
class SursurfEditOnOff(bpy.types.Operator):
    bl_idname = "object.subsurfeditonoff"
    bl_label = "Sursurf Edit On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Subsurf"].show_in_editmode = not bpy.context.object.modifiers["Subsurf"].show_in_editmode
        return {'FINISHED'}  

#Sursurf Cage On/Off
class SursurfCageOnOff(bpy.types.Operator):
    bl_idname = "object.subsurfcageonoff"
    bl_label = "Sursurf Cage On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Subsurf"].show_on_cage = not bpy.context.object.modifiers["Subsurf"].show_on_cage
        return {'FINISHED'}  


#Sursurf Optimal Display
class SursurfOptimalDisplay(bpy.types.Operator):
    bl_idname = "object.subsurfoptimaldisplay"
    bl_label = "Sursurf Optimal Display"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Subsurf"].show_only_control_edges = not bpy.context.object.modifiers["Subsurf"].show_only_control_edges
        return {'FINISHED'}  
    
#######Mirror###########

#Mirror On/Off
class MirrorOnOff(bpy.types.Operator):
    bl_idname = "object.mirroronoff"
    bl_label = "Mirror On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Mirror"].show_viewport = not bpy.context.object.modifiers["Mirror"].show_viewport
        return {'FINISHED'} 
    
#Mirror Edit On/Off
class MirrorEditOnOff(bpy.types.Operator):
    bl_idname = "object.mirroreditonoff"
    bl_label = "Mirror Edit On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Mirror"].show_in_editmode = not bpy.context.object.modifiers["Mirror"].show_in_editmode
        return {'FINISHED'}  

#Mirror Cage On/Off
class MirrorCageOnOff(bpy.types.Operator):
    bl_idname = "object.mirrorcageonoff"
    bl_label = "Mirror Cage On/Off"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Mirror"].show_on_cage = not bpy.context.object.modifiers["Mirror"].show_on_cage
        return {'FINISHED'}  

#Mirror Clipping
class MirrorClipping(bpy.types.Operator):
    bl_idname = "object.mirrorclipping"
    bl_label = "Mirror Clipping"
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        layout = self.layout 
        
        bpy.context.object.modifiers["Mirror"].use_clip = not bpy.context.object.modifiers["Mirror"].use_clip
        return {'FINISHED'} 
                           
#################### Panel ####################    
bpy.types.Scene.retopo_Tools = bpy.props.BoolProperty(default=True)
bpy.types.Scene.retopo_Modifiers = bpy.props.BoolProperty(default=True)
bpy.types.Scene.retopo_Shading = bpy.props.BoolProperty(default=True)
bpy.types.Scene.retopo_Normals = bpy.props.BoolProperty(default=True)

  
class RetopologyTools(bpy.types.Panel): 
    
    bl_label = "Retopology Tools"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'TOOLS'
    bl_category = "Retopology"
  
    def draw(self, context): 
        layout = self.layout 
        
        
        
        if bpy.context.object and bpy.context.object.type == 'MESH':
            if bpy.context.object.select: 
            
                #Tools    
                if context.scene.retopo_Tools==False:
                    layout.prop(context.scene, "retopo_Tools", text="Tools", icon='TRIA_DOWN') 
                    
                    layout.operator("setup.retopomesh", icon = 'UV_FACESEL')
                    layout.operator("polysculpt.retopo", text = "Sculpt Mesh", icon = 'SCULPTMODE_HLT')
                    
                    if bpy.context.object.mode == "EDIT":
                        row = layout.row(align=True)
                        row.operator("freeze_verts.retopo", text = 'Freeze')
                        row.operator("thaw_freeze_verts.retopo", text = 'Thaw')
                        row.operator("show_freeze_verts.retopo", text = 'Show')
                        layout.operator("shrink.update", text = "Shrinkwrap Update", icon = 'MOD_SHRINKWRAP')
                        layout.separator()
                        layout.operator("gpencil.surfsk_add_surface", text="Add Bsurface", icon = 'MOD_DYNAMICPAINT')
                        layout.operator("mesh.looptools_gstretch", text="GStretch", icon='GREASEPENCIL')
                        layout.operator("mesh.laprelax", icon = 'MOD_LATTICE')
                        layout.menu("VIEW3D_MT_edit_mesh_looptools")
                        layout.operator("object.createhole", icon='CLIPUV_DEHLT', text="Create Hole")
                        layout.operator("object.align2x", icon='MOD_WIREFRAME')
                    else: 
                        layout.separator() 
                    
                    box = layout.box()
                    row = box.row()
                    row.prop(context.space_data, "show_occlude_wire")
                    row.prop(context.object, "show_x_ray", text="X-Ray")
                    
                    row = box.row()
                    row.prop(context.space_data, "show_backface_culling", text="BF Culling")
                    row.prop(context.object.data, "show_double_sided")
                    
                    row = box.row()
                    row.prop(context.tool_settings, "use_mesh_automerge", text = "Auto Merge")
                    row.prop(context.space_data, "use_occlude_geometry", text = "Occlude Geo")
                    
                    row = layout.row()
                    box.label("Double Threshold:")
                    
                    row = box.split(percentage=0.5)
                    tool_settings = context.tool_settings
                    row.prop(tool_settings, "double_threshold", text="")
                    row.operator("double.threshold01", text= "0.1")
                    row.operator("double.threshold0001", text= "0.001")
                    col = layout.column(align=True)
                else:
                    layout.prop(context.scene, "retopo_Tools", text="Tools", icon='TRIA_RIGHT')
                    
                #Shading
                if context.scene.retopo_Shading==False:
                    layout.prop(context.scene, "retopo_Shading", text="Shading", icon='TRIA_DOWN')
                    
                    box = layout.box()
                    row = box.row()
                    row.operator("shading.smooth", text="Smooth")
                    row.operator("shading.flat", text="Flat")
                    row.operator("wire.selectedall", text="Wire", icon='WIRE')
                    if bpy.context.object.mode == "EDIT":
                        row = layout.row()
                        row.operator("mesh.flip_normals", icon = 'FILE_REFRESH')
                        row.operator("mesh.normals_make_consistent", icon = 'MATCUBE')
                        row = layout.row()
                        row.operator("wm.context_toggle", text="Show Norms", icon='FACESEL').data_path = "object.data.show_normal_face"
                        row.operator("mesh.remove_doubles",icon='X')
                    else: 
                        layout.separator() 
                    
                    #Matcaps
                    view = context.space_data
                    col = layout.column()
                    if view.viewport_shade == 'SOLID':
                        col.prop(context.space_data, "use_matcap")
                        if view.use_matcap:
                            col.template_icon_view(context.space_data, "matcap_icon")
                    
                       
                else:
                    layout.prop(context.scene, "retopo_Shading", text="Shading", icon='TRIA_RIGHT')
                    
                    
                #Modifiers
                if context.scene.retopo_Modifiers==False:
                    layout.prop(context.scene, "retopo_Modifiers", text="Modifiers", icon='TRIA_DOWN') 
                    #Mirror
                    box = layout.box()
                    row = box.row()
                    is_mirror = False
                    for mode in bpy.context.object.modifiers :
                        if mode.type == 'MIRROR' :
                            is_mirror = True
                    if is_mirror == True :
                        row.operator("object.modifier_remove", text="Del Mirror", icon='X').modifier="Mirror"
                    else:
                        row.operator("add.mirrorobject", icon = 'MOD_MIRROR', text="Add Mirror")
                    
                    row.operator("object.automirror", icon = 'MOD_MIRROR')
                    row = box.row()
                    row.operator("apply.mirror", text="Apply Mirror", icon='FILE_TICK')
                    
                    #On/Off, Edit, Cage
                    row = box.row()
                    for mode in bpy.context.object.modifiers :
                        if mode.type == 'MIRROR' :
                            is_mirror = True
                    if is_mirror == True :

                        #View On/Off
                        if bpy.context.object.modifiers["Mirror"].show_viewport == (True) :
                            row.operator("object.mirroronoff", text="View",icon='RESTRICT_VIEW_OFF')
                        else:
                            row.operator("object.mirroronoff", text="View",icon='VISIBLE_IPO_OFF')   
                        #Edit On/Off
                        if bpy.context.object.modifiers["Mirror"].show_in_editmode == (True) :
                            row.operator("object.mirroreditonoff", text="Edit",icon='EDITMODE_HLT')
                        else:
                            row.operator("object.mirroreditonoff", text="Edit",icon='SNAP_VERTEX') 
                        #Cage On/Off
                        if bpy.context.object.modifiers["Mirror"].show_on_cage == (True) :
                            row.operator("object.mirrorcageonoff", text="Cage",icon='OUTLINER_OB_MESH')
                        else:
                            row.operator("object.mirrorcageonoff", text="Cage",icon='OUTLINER_DATA_MESH') 
                        #Clipping
                        if bpy.context.object.modifiers["Mirror"].use_clip == (True) :
                            row.operator("object.mirrorclipping", text="Clip",icon='UV_EDGESEL')
                        else:
                            row.operator("object.mirrorclipping", text="Clip",icon='SNAP_EDGE') 
                    #Subsurf
                    box = layout.box()
                    row = box.row()
                    is_subsurf = False
                    for mode in bpy.context.object.modifiers :
                        if mode.type == 'SUBSURF' :
                            is_subsurf = True
                    if is_subsurf == True :
                        row.operator("object.removesubsurf2", text="Del Subsurf", icon='X')
                        
                    else :
                        row.operator("object.subsurf2", text="Add Subsurf", icon='MOD_SUBSURF')
                    
                    row.operator("object.applysubsurf", text="Apply Subsurf", icon='FILE_TICK')
                    layout.operator("delete.modifiers", text="Delete Modifiers", icon='X')
                    
                    #On/Off, Edit, Cage
                    row = box.row()
                    for mode in bpy.context.object.modifiers :
                        if mode.type == 'SUBSURF' :
                            is_subsurf = True
                    if is_subsurf == True :

                        #View On/Off
                        if bpy.context.object.modifiers["Subsurf"].show_viewport == (True) :
                            row.operator("object.subsurfonoff", text="View",icon='RESTRICT_VIEW_OFF')
                        else:
                            row.operator("object.subsurfonoff", text="View",icon='VISIBLE_IPO_OFF')   
                        #Edit On/Off
                        if bpy.context.object.modifiers["Subsurf"].show_in_editmode == (True) :
                            row.operator("object.subsurfeditonoff", text="Edit",icon='EDITMODE_HLT')
                        else:
                            row.operator("object.subsurfeditonoff", text="Edit",icon='SNAP_VERTEX') 
                        #Cage On/Off
                        if bpy.context.object.modifiers["Subsurf"].show_on_cage == (True) :
                            row.operator("object.subsurfcageonoff", text="Cage",icon='OUTLINER_OB_MESH')
                        else:
                            row.operator("object.subsurfcageonoff", text="Cage",icon='OUTLINER_DATA_MESH')
                        #Optimal display
                        if bpy.context.object.modifiers["Subsurf"].show_only_control_edges == (True) :
                            row.operator("object.subsurfoptimaldisplay", text="OD",icon='SOLID')
                        else:
                            row.operator("object.subsurfoptimaldisplay", text="OD",icon='WIRE')
                else:
                    layout.prop(context.scene, "retopo_Modifiers", text="Modifiers", icon='TRIA_RIGHT')
        
            else:
                layout.label(icon="ERROR", text="No Reference Selected")    
        else:
            layout.label(icon="ERROR", text="No Reference Selected")

######################
#     Pie Menus      #               
######################

#################### Operators ####################

#Space
class RetopoSpace(bpy.types.Operator):  
    bl_idname = "retopo.space"  
    bl_label = "Retopo Space"  
  
    def execute(self, context):
        bpy.ops.mesh.looptools_space(influence=100, input='selected', interpolation='cubic', lock_x=False, lock_y=False, lock_z=False)
        return {'FINISHED'} 
            
#SCT_Select_Only
class SCTSelectOnly(bpy.types.Operator):  
    bl_idname = "object.sctselectonly"  
    bl_label = "Select Only"  
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.smooth_vertices
        
        props.only_selected_are_affected = not props.only_selected_are_affected
        
        return {'FINISHED'}  

#SCT_Lock_Boundary
class SCTLockBoundary(bpy.types.Operator):  
    bl_idname = "object.sctlockboundary"  
    bl_label = "Lock Boundary Smooth"  
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.smooth_vertices
        
        props.boundary_is_locked = not props.boundary_is_locked
        
        return {'FINISHED'} 

#SCT_Isolate_Selection
class SCTIsolateSelection(bpy.types.Operator):  
    bl_idname = "object.sctisolateselection"  
    bl_label = "SCT_Isolate_Selection"  
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.mesh_brush
        
        props.selection_is_isolated = not props.selection_is_isolated
        
        return {'FINISHED'}  

#SCT_Lock_Boundary_Brush
class SCTLockBoundaryBrush(bpy.types.Operator):  
    bl_idname = "object.sctlockboundarybrush"  
    bl_label = "Lock Boundary Brush"  
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.mesh_brush
        
        props.boundary_is_locked = not props.boundary_is_locked
        
        return {'FINISHED'} 

#SCT_Select_Only_Shrinkwrap
class SCTSelectOnlyShrinkwrap(bpy.types.Operator):  
    bl_idname = "object.sctselectonlyshrinkwrap"  
    bl_label = "Select Only Shrinkwrap"  
    bl_options = {'REGISTER', 'UNDO'}
    
    def execute(self, context):
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.shrinkwrap
        
        props.only_selected_are_affected = not props.only_selected_are_affected
        
        return {'FINISHED'} 

#################### Pie Menus ####################
           
#Pie Retopo - Shift + RMB              
class PieRetopo(Menu):
    bl_idname = "pie.retopo"
    bl_label = "Pie Retopo"

    def draw(self, context):
        layout = self.layout
        pie = layout.menu_pie()
        #4 - LEFT
        pie.operator("mesh.sct_mesh_brush", text="Mesh Brush", icon='BRUSH_DATA')
        #6 - RIGHT
        pie.operator("align.2x0", icon='MOD_WIREFRAME')
        #2 - BOTTOM
        pie.operator("mesh.sct_smooth_vertices", icon = 'MOD_LATTICE')
        #8 - TOP
        pie.operator("gpencil.surfsk_add_surface", text="Add Bsurface", icon = 'MOD_DYNAMICPAINT')
        #7 - TOP - LEFT 
        box = pie.split().column()
        row = box.split(align=True)
        row.operator("setup.retopomesh", icon = 'UV_FACESEL')
        row = box.split(align=True)
        row.operator("freeze_verts.retopo", text = 'Freeze')
        row.operator("thaw_freeze_verts.retopo", text = 'Thaw')
        row.operator("show_freeze_verts.retopo", text = 'Show')
        #9 - TOP - RIGHT
        box = pie.split().column()
        box.operator("shrink.update", text = "Ice Shrink Update", icon = 'MOD_SHRINKWRAP')   
        box.operator("mesh.sct_shrinkwrap", text = "SCT Shrink Update", icon = 'MOD_SHRINKWRAP') 
        #1 - BOTTOM - LEFT
        box = pie.split().column()
        row = box.split(align=True)
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.mesh_brush
        #Isolate Selection
        if props.selection_is_isolated == False:
            row.operator("object.sctisolateselection",text="Brush IS", icon = 'CHECKBOX_DEHLT')
        else :
            row.operator("object.sctisolateselection",text="Brush IS", icon = 'CHECKBOX_HLT') 
        #Lock Boundary 
        if props.boundary_is_locked == False:
            row.operator("object.sctlockboundarybrush",text="Brush LB", icon = 'CHECKBOX_DEHLT')
        else :
            row.operator("object.sctlockboundarybrush",text="Brush LB", icon = 'CHECKBOX_HLT') 
        
        row = box.row(align=True)
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.smooth_vertices
        #Select Only
        if props.only_selected_are_affected == False:
            row.operator("object.sctselectonly",text="Smooth SO", icon = 'CHECKBOX_DEHLT')
        else :
            row.operator("object.sctselectonly",text="Smooth SO", icon = 'CHECKBOX_HLT') 
        #Lock Boundary    
        if props.boundary_is_locked == False:
            row.operator("object.sctlockboundary",text="Smooth LB", icon = 'CHECKBOX_DEHLT')
        else :
            row.operator("object.sctlockboundary",text="Smooth LB", icon = 'CHECKBOX_HLT')   
        row = box.split(align=True)    
        addon = bpy.context.user_preferences.addons["surface-constraint-tools-master"]
        props = addon.preferences.shrinkwrap   
        #Only Selected Shrinkwrap   
        if props.only_selected_are_affected == False:
            row.operator("object.sctselectonlyshrinkwrap",text="Shrinkwrap SO", icon = 'CHECKBOX_DEHLT')
        else :
            row.operator("object.sctselectonlyshrinkwrap",text="Shrinkwrap SO", icon = 'CHECKBOX_HLT') 
        #3 - BOTTOM - RIGHT
        pie.operator("retopo.space", icon='ALIGN', text="Space")        

#################### Register/Unregister/Keymap ####################
        
addon_keymaps = []        
  
def register(): 
    bpy.utils.register_class(WazouRetopoTools)
    bpy.utils.register_class(RetopologyTools)
    bpy.utils.register_class(AlignToX)
    bpy.utils.register_class(LapRelax)
    bpy.utils.register_class(DoubleThreshold01)
    bpy.utils.register_class(DoubleThreshold0001)
    bpy.utils.register_class(SetupRetopoMesh)
    bpy.utils.register_class(WireSelectedAll)
    bpy.utils.register_class(ShadingSmooth)
    bpy.utils.register_class(ShadingFlat)
    bpy.utils.register_class(CreateHole)
    bpy.utils.register_class(DeleteModifiers)
    bpy.utils.register_class(AddMirrorObject)
    bpy.utils.register_class(ApplyMirror)
    bpy.utils.register_class(SubSurf2)
    bpy.utils.register_class(ApplySubSurf)
    bpy.utils.register_class(RemoveSubSurf2)
    bpy.utils.register_class(SursurfOnOff) 
    bpy.utils.register_class(SursurfEditOnOff) 
    bpy.utils.register_class(SursurfCageOnOff) 
    bpy.utils.register_class(SursurfOptimalDisplay) 
    bpy.utils.register_class(MirrorOnOff) 
    bpy.utils.register_class(MirrorEditOnOff) 
    bpy.utils.register_class(MirrorCageOnOff)
    bpy.utils.register_class(MirrorClipping)
    #Pie Menus
    bpy.utils.register_class(PieRetopo)
    bpy.utils.register_class(RetopoSpace)
    bpy.utils.register_class(SCTLockBoundary)
    bpy.utils.register_class(SCTIsolateSelection)
    bpy.utils.register_class(SCTSelectOnly)
    bpy.utils.register_class(SCTLockBoundaryBrush)
    bpy.utils.register_class(SCTSelectOnlyShrinkwrap)
    

# Keympa Config   
    
    wm = bpy.context.window_manager
    
    if wm.keyconfigs.addon:
        
        #Retopo
        km = wm.keyconfigs.addon.keymaps.new(name = '3D View Generic', space_type = 'VIEW_3D')
        kmi = km.keymap_items.new('wm.call_menu_pie', 'RIGHTMOUSE', 'PRESS', shift=True)
        kmi.properties.name = "pie.retopo"    

        addon_keymaps.append(km)
    
def unregister(): 
    bpy.utils.register_class(WazouRetopoTools)
    bpy.utils.unregister_class(RetopologyTools) 
    bpy.utils.unregister_class(AlignToX)
    bpy.utils.unregister_class(LapRelax)
    bpy.utils.unregister_class(DoubleThreshold01)
    bpy.utils.unregister_class(DoubleThreshold0001)
    bpy.utils.unregister_class(SetupRetopoMesh)
    bpy.utils.unregister_class(WireSelectedAll)
    bpy.utils.unregister_class(ShadingSmooth)
    bpy.utils.unregister_class(ShadingFlat)
    bpy.utils.unregister_class(CreateHole)
    bpy.utils.unregister_class(DeleteModifiers)
    bpy.utils.unregister_class(AddMirrorObject)
    bpy.utils.unregister_class(ApplyMirror)
    bpy.utils.unregister_class(SubSurf2)
    bpy.utils.unregister_class(ApplySubSurf)
    bpy.utils.unregister_class(RemoveSubSurf2)
    bpy.utils.unregister_class(SursurfOnOff) 
    bpy.utils.unregister_class(SursurfEditOnOff) 
    bpy.utils.unregister_class(SursurfCageOnOff)
    bpy.utils.unregister_class(SursurfOptimalDisplay)
    bpy.utils.unregister_class(MirrorOnOff) 
    bpy.utils.unregister_class(MirrorEditOnOff) 
    bpy.utils.unregister_class(MirrorCageOnOff)
    bpy.utils.unregister_class(MirrorClipping)
    # Pie Menus
    bpy.utils.unregister_class(PieRetopo)
    bpy.utils.unregister_class(RetopoSpace)
    bpy.utils.unregister_class(SCTLockBoundary)
    bpy.utils.unregister_class(SCTIsolateSelection)
    bpy.utils.unregister_class(SCTSelectOnly)
    bpy.utils.unregister_class(SCTLockBoundaryBrush)
    bpy.utils.unregister_class(SCTSelectOnlyShrinkwrap)
    
    wm = bpy.context.window_manager

    if wm.keyconfigs.addon:
        for km in addon_keymaps:
            for kmi in km.keymap_items:
                km.keymap_items.remove(kmi)

            wm.keyconfigs.addon.keymaps.remove(km)

    # clear the list
    del addon_keymaps[:]
    
if __name__ == "__main__": 
    register() 










