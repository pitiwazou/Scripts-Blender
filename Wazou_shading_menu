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

############# Add-on description (use by Blender)

bl_info = {
    "name": "Shading_menu",
    "description": 'Add a menu with shading options in the 3Dview',
    "author": "Cedric Lepiller, Lapineige",
    "version": (0, 1, 0),
    "blender": (2, 70, 0),
    "location": "View3D > Header > Shading_menu",
    "warning": "", # used for warning icon and text in addons panel
    "wiki_url": "",
    "tracker_url": "http://www.pitiwazou.com",
    "category": "3D View"}

##############

import bpy
from bpy.props import IntProperty, FloatProperty, BoolProperty

class ShadingMenuCACUserPrefs(bpy.types.AddonPreferences):
    """Creates the tools in a Panel, in the scene context of the properties editor"""
    bl_idname = __name__

    bpy.types.Scene.Enable_ShadinMenu_Tab_1 = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.Enable_ShadinMenu_Tab_2 = bpy.props.BoolProperty(default=False)
   

    def draw(self, context):
        layout = self.layout

        layout.prop(context.scene, "Enable_ShadinMenu_Tab_1", text="Info", icon="QUESTION")
        if context.scene.Enable_ShadinMenu_Tab_1:
            row = layout.row()
            layout.label(text="This Addons is a Menu with shading options")
            
        layout.prop(context.scene, "Enable_ShadinMenu_Tab_2", text="URL's", icon="URL")   
        if context.scene.Enable_ShadinMenu_Tab_2:
            row = layout.row()
            row.operator("wm.url_open", text="Pitiwazou.com").url = "http://www.pitiwazou.com/"
            row.operator("wm.url_open", text="Wazou's Ghitub").url = "https://github.com/pitiwazou/Scripts-Blender"
            row.operator("wm.url_open", text="BlenderLounge Forum ").url = "http://blenderlounge.fr/forum/"
            
        return {'FINISHED'}


#Solid All
class SolidAll(bpy.types.Operator):

    bl_idname = "object.solid_all"
    bl_label = "Solid All"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    
    def execute(self, context):
        for obj in bpy.data.objects: 
            if obj.draw_type == 'BOUNDS': 
                obj.draw_type = 'SOLID'
            elif obj.draw_type == 'SOLID':
                obj.draw_type = 'SOLID' 
            else:                       
                obj.draw_type = 'BOUNDS'
       
        return {'FINISHED'}

#Bounds Unselected
class BoundsUnselected(bpy.types.Operator):

    bl_idname = "object.draw_bu"
    bl_label = "bounds"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    
    def execute(self, context):
        for obj in bpy.data.objects: 
            if not obj.select:
                
                if obj.draw_type == 'SOLID':
                    obj.draw_type = 'BOUNDS'
                    obj.show_all_edges = False
                    obj.show_wire = False
                   
                elif obj.draw_type == 'BOUNDS':
                    obj.draw_type = 'BOUNDS'
              
                else:                       
                    obj.draw_type = 'SOLID'
        
        for obj in bpy.data.objects:
            if obj.select:
                obj.draw_type = 'SOLID'

        return {'FINISHED'}

#auto smooth Menu
class AutoSmoothMenu(bpy.types.Menu):
    bl_idname = "auto.smooth_menu"
    bl_label = "Auto_Smooth_Menu"

    def draw(self, context):
        layout = self.layout
        
        layout.operator("auto.smooth_30", text="Auto Smooth 30", icon='MESH_DATA')
        layout.operator("auto.smooth_45", text="Auto Smooth 45", icon='MESH_DATA')
        layout.operator("auto.smooth_89", text="Auto Smooth 89", icon='MESH_DATA')

#AutoSmooth_89
class AutoSmooth89(bpy.types.Operator):
    bl_idname = "auto.smooth_89"
    bl_label = "Auto Smooth"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
        
    def execute(self, context):
        bpy.context.object.data.auto_smooth_angle = 1.55334
        return {'FINISHED'}

#AutoSmooth_30
class AutoSmooth30(bpy.types.Operator):
    bl_idname = "auto.smooth_30"
    bl_label = "Auto Smooth"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
        
    def execute(self, context):
        bpy.context.object.data.auto_smooth_angle = 0.523599
        return {'FINISHED'}

#AutoSmooth_45
class AutoSmooth45(bpy.types.Operator):
    bl_idname = "auto.smooth_45"
    bl_label = "Auto Smooth"

    @classmethod
    def poll(cls, context):
        return context.active_object is not None
        
    def execute(self, context):
        bpy.context.object.data.auto_smooth_angle = 0.785398
        return {'FINISHED'}

#Wire on all objects
class WireAll(bpy.types.Operator):
    bl_idname = "object.wireall"
    bl_label = "Wire All"

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

#Grid show/hide with axes
class ToogleGridAxis(bpy.types.Operator):
    bl_idname = "scene.tooglegridaxis"
    bl_label = "Toogle Grid and Axis in 3D view"

    def execute(self, context):
        bpy.context.space_data.show_axis_y = not bpy.context.space_data.show_axis_y
        bpy.context.space_data.show_axis_x = not bpy.context.space_data.show_axis_x
        bpy.context.space_data.show_floor = not bpy.context.space_data.show_floor
        return {'FINISHED'}

#Shading Smooth
class ShadingSmooth(bpy.types.Operator):
    bl_idname = "shading.smooth"
    bl_label = "Shading Smooth"
    
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
    
    def execute(self, context):
        if bpy.context.object.mode == "OBJECT":
            bpy.ops.object.shade_flat()
            
        elif bpy.context.object.mode == "EDIT":
            bpy.ops.object.mode_set(mode = 'OBJECT')
            bpy.ops.object.shade_flat()
            bpy.ops.object.mode_set(mode = 'EDIT')
        return {'FINISHED'} 

#shading Menu
class Shading(bpy.types.Menu):
    bl_label = "Shading"
    
    def draw(self, context):
        layout = self.layout     
        obj = context.object
        view = context.space_data
        
        col = layout.column() 
        if context.space_data.show_floor:
            col.operator("scene.tooglegridaxis", text="Grid", icon = 'CHECKBOX_HLT')
        else:
            col.operator("scene.tooglegridaxis", text="Grid", icon = 'CHECKBOX_DEHLT')  
        
        col = layout.column()
        col.prop(view, "use_matcap")
        
        col = layout.column()
        col.prop(view, "lock_camera")
        
        col = layout.column()
        col.prop(view, "show_only_render")
        
        layout.separator()
        
        layout.menu("auto.smooth_menu", text="Auto Smooth Values", icon='MESH_DATA')
        col = layout.column() 
        col.prop(obj.data, "use_auto_smooth")
        
        col = layout.column() 
        col.prop(obj.data, "show_normal_face", text = "Show Normals")
        
        layout.operator("shading.flat", icon = 'ALIASED')
        layout.operator("shading.smooth", icon = 'ANTIALIASED')
        
        layout.separator()
        
        col = layout.column() 
        col.prop(obj.data, "show_double_sided")
        
        col = layout.column() 
        col.prop(view, "show_backface_culling")
        
        col = layout.column() 
        col.prop(view, "use_occlude_geometry", text="Limit to Visible")
        
        col = layout.column() 
        col.prop(view, "show_occlude_wire")
        
        col = layout.column() 
        col.prop(obj, "show_x_ray", text="X-Ray")
        
        layout.separator()
        
        layout.operator("object.solid_all", text="Solid All", icon='MESH_CUBE')
        layout.operator("object.draw_bu", text="Bounds Unselected", icon='GROUP_VERTEX')
        layout.operator("object.wire_selected", text="Wire Selected", icon='WIRE')
        layout.operator("object.wireall", text="Wire All", icon='WIRE')


def view3d_Shading_menu(self, context):
    layout = self.layout
    layout.menu("Shading")
    
   

def register():
    bpy.types.VIEW3D_HT_header.append(view3d_Shading_menu)
    bpy.utils.register_class(Shading)
    bpy.utils.register_class(ToogleGridAxis) 
    bpy.utils.register_class(WireAll)
    bpy.utils.register_class(SolidAll)
    bpy.utils.register_class(BoundsUnselected)
    bpy.utils.register_class(AutoSmoothMenu)
    bpy.utils.register_class(AutoSmooth89)
    bpy.utils.register_class(AutoSmooth30)
    bpy.utils.register_class(AutoSmooth45)
    bpy.utils.register_class(ShadingFlat)
    bpy.utils.register_class(ShadingSmooth)
    bpy.utils.register_class(ShadingMenuCACUserPrefs)
    
    
def unregister():
    bpy.types.VIEW3D_HT_header.remove(view3d_Shading_menu)
    bpy.utils.unregister_class(Shading)
    bpy.utils.unregister_class(ToogleGridAxis)
    bpy.utils.unregister_class(WireAll)
    bpy.utils.unregister_class(SolidAll)
    bpy.utils.unregister_class(BoundsUnselected)
    bpy.utils.unregister_class(AutoSmoothMenu)
    bpy.utils.unregister_class(AutoSmooth89)
    bpy.utils.unregister_class(AutoSmooth30)
    bpy.utils.unregister_class(AutoSmooth45)
    bpy.utils.unregister_class(ShadingFlat)
    bpy.utils.unregister_class(ShadingSmooth)
    bpy.utils.unregister_class(ShadingMenuCACUserPrefs)
  
if __name__ == "__main__":
    register()










