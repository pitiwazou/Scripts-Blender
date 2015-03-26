bl_info = {
    "name": "Addon Create Hole",
    "author": "Cédric Lepiller",
    "version": (0, 1, 1),
    "blender": (2, 74, 0),
    "location": "View3D > Tools",
    "description": "This Operator create a hole on a selection",
    "warning": "",
    "wiki_url": "",
    "category": "Tools"}
     
import bpy
 
#Create Hole
class CreateHole(bpy.types.Operator):                  
    """This Operator create a hole on a selection"""                  
    bl_idname = "object.createhole"                    
    bl_label = "Create Hole"       
 
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
     
##################################
 
#Class Panel
class PanelCreateHole(bpy.types.Panel):
    bl_label = "Create Hole"
    bl_idname = "OBJECT_PT_Create_hole"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'TOOLS'
    bl_category = "Tools"
    bl_context = "mesh_edit"
       
 
    def draw(self, context):
        layout = self.layout
        
        layout.operator("object.createhole", icon='CLIPUV_DEHLT')
 
 
def register():
    bpy.utils.register_class(PanelCreateHole)
    bpy.utils.register_class(CreateHole)
 
def unregister():
    bpy.utils.unregister_class(PanelCreateHole)
    bpy.utils.unregister_class(CreateHole)
 
if __name__ == "__main__":
    register()
