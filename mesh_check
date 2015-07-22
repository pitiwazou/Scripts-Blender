bl_info = {
    "name": "Mesh check",
    "author": "Clarkx, Cedric Lepiller, CoDEmanX, Pistiwique",
    "version": (0, 1, 1),
    "blender": (2, 75, 0),
    "description": "Custom Menu to show faces, tris, Ngons on the mesh",
    "category": "3D View",}
    
import bpy
import bmesh
from bpy.props import EnumProperty

class WazouPieMenuPrefs(bpy.types.AddonPreferences):
    """Check your meshes for Ngons, Tris"""
    bl_idname = __name__

    bpy.types.Scene.Enable_Tab_01 = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.Enable_Tab_02 = bpy.props.BoolProperty(default=False)

    def draw(self, context):
        layout = self.layout
        
        layout.prop(context.scene, "Enable_Tab_01", text="info", icon="INFO")   
        if context.scene.Enable_Tab_01:
            row = layout.row()
            layout.label(text="This Addon allow you to check your meshes and select Ngons and Tris")
            layout.label(text="This Addon is placed in the N panel / Shading")

        layout.prop(context.scene, "Enable_Tab_02", text="URL's", icon="URL")   
        if context.scene.Enable_Tab_02:
            row = layout.row()
            row.operator("wm.url_open", text="Pitiwazou.com").url = "http://www.pitiwazou.com/"
            row.operator("wm.url_open", text="Wazou's Ghitub").url = "https://github.com/pitiwazou/Scripts-Blender"
            row.operator("wm.url_open", text="BlenderLounge Forum ").url = "http://blenderlounge.fr/forum/"
 

mesh_check_handle = []
display_color = [False]  


def Setup_Scene():
    bpy.ops.object.mode_set(mode='OBJECT')
    for slots in bpy.context.active_object.material_slots:
        bpy.ops.object.material_slot_remove() # remove all materials slots   
    bpy.ops.object.mode_set(mode='EDIT')

   
def Create_Mat():
    m_check = bpy.context.window_manager.m_check
    if bpy.context.space_data.use_matcap:
        m_check.meshcheck_matcap = True
        bpy.context.space_data.use_matcap = False
                            
    # create new material        
    mat_A = bpy.data.materials.new("Quads")
    mat_A.diffuse_color = (0.865, 0.865, 0.865)

    mat_B = bpy.data.materials.new("Ngons")
    mat_B.diffuse_color = (1, 0, 0)
    
    mat_C = bpy.data.materials.new("Tris")
    mat_C.diffuse_color = (1, 1, 0)
    
    ob = bpy.context.active_object
    me = ob.data
    mat_list = [mat_A, mat_B, mat_C]
    for mat in mat_list:
        me.materials.append(mat) # assign materials

       
def Restore_Mat():
    Setup_Scene()
    mat_A = bpy.data.materials['Quads']

    mat_B = bpy.data.materials['Ngons']
    
    mat_C = bpy.data.materials['Tris']
    
    ob = bpy.context.active_object
    me = ob.data
    mat_list = [mat_A, mat_B, mat_C]
    for mat in mat_list:
        me.materials.append(mat)


class AddMaterials(bpy.types.Operator):
    bl_idname = "object.add_materials"
    bl_label = "Add materials"
    
    @classmethod
    def poll(cls, context):
        return context.object is not None and context.object.type == 'MESH'
    
    def execute(sefl, context):        
        m_check = context.window_manager.m_check  
        m_check.meshcheck_shade = bpy.context.space_data.viewport_shade
        bpy.context.space_data.viewport_shade = 'SOLID'         
        if not bpy.data.materials:
            if context.object.mode == 'OBJECT':
                bpy.ops.object.mode_set(mode='EDIT')
                Create_Mat() 
            elif context.object.mode == 'EDIT':
                Create_Mat()
            m_check.meshcheck_enabled = True           
            return {'FINISHED'}
        else:
            if bpy.context.object.active_material:
                for mat_slots in bpy.context.active_object.material_slots:
                    m_check.meshcheck_save_mat.append(mat_slots.name)
                        
            ref_list = ['Quads', 'Ngons', 'Tris']            
            mat_list = []
            for mat in bpy.data.materials:
                mat_list.append(mat.name)
            for ref in ref_list:
                if ref in mat_list:
                    if bpy.context.space_data.use_matcap:
                        m_check.meshckeck_matcap = True
                        bpy.context.space_data.use_matcap = False 
                    else:
                        m_check.meshckeck_matcap = False                    
                    Restore_Mat()
                    m_check.meshcheck_enabled = True           
                    return {'FINISHED'}
                else:
                    Setup_Scene() 
                    Create_Mat()
                    m_check.meshcheck_enabled = True           
                    return {'FINISHED'}


class RemoveMaterials(bpy.types.Operator):
    bl_idname = "object.remove_materials"
    bl_label = "Remove materials"
    
    @classmethod
    def poll(cls, context):
        return context.active_object.type == 'MESH'
    
    def execute(self, context):
        m_check = context.window_manager.m_check       
        if bpy.context.object.mode == 'OBJECT':
            for slots in bpy.context.active_object.material_slots:
                bpy.ops.object.material_slot_remove()
            if len(m_check.meshcheck_save_mat) > 0:
                ob = bpy.context.active_object
                me = ob.data
                
                for mat in m_check.meshcheck_save_mat:
                    mat_list = bpy.data.materials[mat]
                    me.materials.append(mat_list)

                m_check.meshcheck_save_mat[:] = [] # clean the list
            if m_check.meshcheck_matcap:
                bpy.context.space_data.use_matcap = True 
        elif bpy.context.object.mode == 'EDIT':                       
            bpy.ops.object.mode_set(mode='OBJECT')
            for slots in bpy.context.active_object.material_slots:
                bpy.ops.object.material_slot_remove()
            if len(m_check.meshcheck_save_mat) > 0:
                ob = bpy.context.active_object
                me = ob.data
                
                for mat in m_check.meshcheck_save_mat:
                    mat_list = bpy.data.materials[mat]
                    me.materials.append(mat_list)

                m_check.meshcheck_save_mat[:] = [] # clean the list
            if m_check.meshcheck_matcap:
                bpy.context.space_data.use_matcap = True   
            bpy.ops.object.mode_set(mode='EDIT')
        
        bpy.context.space_data.viewport_shade = m_check.meshcheck_shade
        m_check.meshcheck_enabled = False        
        return {'FINISHED'}
                
                
def mesh_check_display_color_callback():    
    obj = bpy.context.object
    if obj and obj.type == 'MESH' and obj.mode == 'EDIT':         
        if display_color[0]:              
            ob = bpy.context.object
            me = ob.data
            bm = bmesh.from_edit_mesh(me)
            
            quads = tris = ngons = 0
            ngons_to_tris = 0            
            verts = len(bm.verts)
            faces = len(bm.faces)
            
            for f in bm.faces:                
                v = len(f.verts)
                if v == 3: # tris
                    f.material_index = 2
                elif v == 4: # quads
                    f.material_index = 0    
                elif v > 4: # ngons 
                    f.material_index = 1
            
            bmesh.update_edit_mesh(me)
        
          
def UpdateFacesColor(self, context):
    if self.meshcheck_enabled:
        display_color[0] = True
        return   
    display_color[0] = False    
        
         
class MeshCheckCollectionGroup(bpy.types.PropertyGroup):
    meshcheck_use = bpy.props.BoolProperty(
        name="Mesh check",
        description="Display mesh check tools",
        default=False)
    meshcheck_enabled = bpy.props.BoolProperty(
        name="Mesh check enabled",
        description="Display faces color",
        default=False, 
        update=UpdateFacesColor)
    meshcheck_shade = bpy.props.StringProperty()
    meshcheck_matcap = bpy.props.BoolProperty(
        name="Mesh check matcap",
        description="Define if matcap enabled or disabled",
        default=False)    
    meshcheck_save_mat = []


class DATA_OP_facetype_select(bpy.types.Operator):
    """Select all faces of a certain type"""
    bl_idname = "data.facetype_select"
    bl_label = "Select by face type"
    bl_options = {'REGISTER', 'UNDO'}
    select_mode = bpy.props.StringProperty()   
    face_type = EnumProperty(name="Select faces:",
                             items = (("3","Triangles","Faces made up of 3 vertices"),
                                      ("5","Ngons","Faces made up of 5 and more vertices")),
                             default = "5")
    @classmethod
    def poll(cls, context):
        return context.object is not None and context.object.type == 'MESH'

    def execute(self, context):
        bpy.ops.object.mode_set(mode='EDIT')
        bpy.ops.mesh.select_all(action='DESELECT')
        if tuple(bpy.context.tool_settings.mesh_select_mode) == (True, False, False):
            self.select_mode = "VERT"
        elif tuple(bpy.context.tool_settings.mesh_select_mode) == (False, True, False):
            self.select_mode = "EDGE"
        elif tuple(bpy.context.tool_settings.mesh_select_mode) == (False, False, True):
            self.select_mode = "FACE"
        context.tool_settings.mesh_select_mode=(False, False, True)
        if self.face_type == "3":
            bpy.ops.mesh.select_face_by_sides(number=3, type='EQUAL')
            bpy.ops.mesh.select_mode(type=self.select_mode)
        else:
            bpy.ops.mesh.select_face_by_sides(number=4, type='GREATER')
            bpy.ops.mesh.select_mode(type=self.select_mode)        
        return {'FINISHED'}        
    
    
def DisplayMeshCheckPanel(self, context):
    layout = self.layout    
    m_check = context.window_manager.m_check
    
    if bpy.context.object and bpy.context.object.type == 'MESH':
        layout.prop(m_check, "meshcheck_use")
        
        if m_check.meshcheck_use:
            split = layout.split(percentage=0.1)
            split.separator()
            if m_check.meshcheck_enabled:
                split.operator("object.remove_materials", text="Hidde color", icon='RESTRICT_VIEW_OFF')
            else:
                split.operator("object.add_materials", text="Display color", icon='COLOR') 
            split = layout.split(percentage=0.1)
            split.separator()
            split2 = split.split(align=True)
            split2.operator("data.facetype_select", text="Ngons").face_type = "5"
            split2.operator("data.facetype_select", text="Tris").face_type = "3"
            if bpy.context.object.mode == 'EDIT':
                    obj = bpy.context.object
                    me = obj.data
                    bm = bmesh.from_edit_mesh(me)
                    	
                    info_str = ""
                    tris = ngons = 0

                    for f in bm.faces:
                
                        v = len(f.verts)
                        if v == 3:
                            tris += 1
                        elif v > 4:
                            ngons += 1
                    
                    bmesh.update_edit_mesh(me)
                    info_str = "  Ngons: %i   Tris: %i" % (ngons, tris)
                    
                    split = layout.split(percentage=0.1)
                    split.separator()
                    split.label(info_str, icon='MESH_DATA')       
        
    
def register():
    bpy.utils.register_module(__name__)
    bpy.types.WindowManager.m_check = bpy.props.PointerProperty(
        type=MeshCheckCollectionGroup)
    bpy.types.VIEW3D_PT_view3d_shading.append(DisplayMeshCheckPanel)
    if mesh_check_handle:
        bpy.types.SpaceView3D.draw_handler_remove(mesh_check_handle[0], 'WINDOW')
    mesh_check_handle[:] = [bpy.types.SpaceView3D.draw_handler_add(mesh_check_display_color_callback, (), 'WINDOW', 'POST_VIEW')]
    
def unregister():
    bpy.types.VIEW3D_PT_view3d_shading.remove(DisplayMeshCheckPanel)
    bpy.utils.unregister_module(__name__)
    del bpy.types.WindowManager.m_check
    if mesh_check_handle:
        bpy.types.SpaceView3D.draw_handler_remove(mesh_check_handle[0], 'WINDOW')
        mesh_check_handle[:] = []

if __name__ == "__main__":
    register()
