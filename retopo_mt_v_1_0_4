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
    "name": "Retopo MT V1.0.4",
    "category": "Mesh",
    "author": "Will Souloumiac aka Pixivore, Cédric Lepiller aka Pitiwazou, Blenderlounge",
    "version": (1,0,4),
    "blender": (2, 77, 0),
    "location": "Editmode > Ctrl+Shift+Alt+X",
    "description": "Multiple tools for retopology.",
}




import bpy
import bpy_extras
from bpy_extras import view3d_utils
import bgl
import bmesh
import blf
import copy
import sys, os
from mathutils import *
from math import *
from bpy.props import IntProperty, FloatProperty
from bpy_extras.object_utils import AddObjectHelper, object_data_add
#########################################################################################################

############### Prefs ########################

#Class Prefs
class RetopoMTPrefs(bpy.types.AddonPreferences):
    bl_idname = __name__

    bpy.types.Scene.Enable_Tab_01 = bpy.props.BoolProperty(default=False)
    bpy.types.Scene.Enable_Tab_02 = bpy.props.BoolProperty(default=False)
    

    def draw(self, context):
        layout = self.layout

        layout.prop(context.scene, "Enable_Tab_01", text="Info", icon="QUESTION")
        if context.scene.Enable_Tab_01:
            row = layout.row()
            layout.label(text="This Addon helps you to make your retopo !")
            layout.label(text="It's an alpha version, so it can crash, think to save your work as soon as possible")
            layout.label(text="Don't hesitate to give your feeling and bug on the forums, Blender artist and Blenderlounge")
            
        layout.prop(context.scene, "Enable_Tab_02", text="URL's", icon="URL")   
        if context.scene.Enable_Tab_02:
            row = layout.row()
            row.operator("wm.url_open", text="pixivores.com").url = "http://pixivores.com"
            row.operator("wm.url_open", text="Pitiwazou.com").url = "http://www.pitiwazou.com"
            row.operator("wm.url_open", text="Wazou's Ghitub").url = "https://github.com/pitiwazou/Scripts-Blender"
            row.operator("wm.url_open", text="BlenderLounge Forum ").url = "http://blenderlounge.fr/forum/"

#########################################################################################################
global g_wire_edit_color
global g_vertex_color
global g_vertex_sel_color
global g_vertex_size
global g_face_color
global g_face_sel_color

global g_Cursor
global g_CutIcon
global g_SizeIcon
global g_Curve
global g_rObject

global g_bDrawing
global g_bPtMoving
global g_bFaceMoving
global g_bMerge
global g_bFaceCreating
global g_bBSurface
global g_bCurveCurrent

global g_BSCurve
global g_BSVs

global g_CTCurve
global g_CTNCurve
global g_bCTRetopo
global g_CTLines
global g_CTNLines
global g_Inverse
global g_CTObj

global g_SVScene
global g_SVregion 
global g_SVrv3d

global g_Selection

global g_hit
global g_normal
global g_faceIndex

global g_mouse_x
global g_mouse_y
global g_bLeftBM
global g_bRightBM

global g_CTRL
global g_SHIFT
global g_ALT
global g_SPACE

global g_region
global g_rv3d

global g_PosCutIcon
global g_PosCutCTIcon
global g_PosSizeIcon
global g_PosSize2d
global g_PosCut2d

global g_bAutoMerge

global g_SavLens
global g_SavPersp
#########################################################################################################


#########################################################################################################
class cCursor:
    def __init__(self, _AngleStep, _Radius):
        # Variables pour le curseur            
        self.CircleListRaw = []
        self.CLR_C = []
        self.CurLoc = Vector((0.0, 0.0, 0.0))
        self.Create(_AngleStep, _Radius)

        self.bHide = False
        self.bBrushSize = False
        self.bCutSize = False
        self.bPointMove = False

#        self.BRadius = 1.0
        self.BRadius = _Radius * 20.0
        self.BSize = 6.0
        self.DRadius = 1.0* 0.65
        
        self.oldmx = 0
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def Hide(self):
        self.bHide = True
        # Hide cursor
        bpy.context.window.cursor_modal_set("DEFAULT")
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Show(self):
        self.bHide = False
        # Hide cursor
        bpy.context.window.cursor_modal_set("NONE")
    #---------------------------------------------------------------------------------------------------
        
    
    #---------------------------------------------------------------------------------------------------
    def CutSize(self, event):
        self.bCutSize = True
        self.oldmx = event.mouse_region_x
    #---------------------------------------------------------------------------------------------------
        
        
    #---------------------------------------------------------------------------------------------------
    def BrushSize(self, event):
        self.bBrushSize = True
        self.oldmx = event.mouse_region_x
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Create(self, _AngleStep, _radius):
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
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def RBenVe(self, Object, Dir):
        ObjectV = Object.normalized()
        DirV = Dir.normalized()
        cosTheta = ObjectV.dot(DirV)
        rotationAxis = Vector((0.0, 0.0, 0.0))
        if (cosTheta < -1 + 0.001):
            v = Vector((0.0, 1.0, 0.0))
            rotationAxis = ObjectV.cross(v)
            rotationAxis = rotationAxis.normalized()
            q = Quaternion()
            q.w = 0.0
            q.x = rotationAxis.x
            q.y = rotationAxis.y
            q.z = rotationAxis.z
            return q
        rotationAxis = ObjectV.cross(DirV);
        s = sqrt((1.0 + cosTheta) * 2.0);
        invs = 1 / s;
        q = Quaternion()
        q.w = s * 0.5
        q.x = rotationAxis.x * invs
        q.y = rotationAxis.y * invs
        q.z = rotationAxis.z * invs
        return q
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Update(self):
        global g_normal
        global g_hit
        global g_mouse_x
        
        NormalObject = Vector((0.0, 0.0, 1.0))
        if(g_normal == None):
            return  
        qRot = self.RBenVe(NormalObject, g_normal)

        if(qRot != None):
            self.CLR_C.clear()
            vc = Vector()        
            idx = 0
            for i in range(int(len(self.CircleListRaw)/3)):
                vc.x = self.CircleListRaw[idx * 3]
                vc.y = self.CircleListRaw[idx * 3  + 1]
                vc.z = self.CircleListRaw[idx * 3 + 2]
                vc = qRot * vc
                self.CLR_C.append(vc.x)
                self.CLR_C.append(vc.y)
                self.CLR_C.append(vc.z)
                idx += 1

        if((self.bBrushSize == False) and (self.bCutSize == False)):
            self.CurLoc = g_hit
            
        if(self.bBrushSize == True):
            if(self.BRadius >= 0.5):
                self.BRadius += (g_mouse_x - self.oldmx) * 0.05
                if(self.BRadius < 0.5):
                    self.BRadius = 0.5
            self.oldmx = g_mouse_x
        elif(self.bCutSize == True):
            if(self.BSize >= 1):
                self.DRadius += (g_mouse_x - self.oldmx) * 0.05
                if((g_mouse_x - self.oldmx) > 0):
                    self.BSize += 1
                else:
                    self.BSize -= 1
                    if(self.BSize < 1):
                        self.BSize = 1
            if(self.DRadius < 0.05):
                self.DRadius = 0.05
            self.oldmx = g_mouse_x
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def RotAndPos(self, hit, normal):
        NormalObject = Vector((0.0, 0.0, 1.0))
        qRot = self.RBenVe(NormalObject, normal)

        if(qRot != None):
            self.CLR_C.clear()
            self.CurLoc = hit
            vc = Vector()        
            idx = 0
            for i in range(int(len(self.CircleListRaw)/3)):
                vc.x = self.CircleListRaw[idx * 3]
                vc.y = self.CircleListRaw[idx * 3  + 1]
                vc.z = self.CircleListRaw[idx * 3 + 2]
                vc = qRot * vc
                self.CLR_C.append(vc.x)
                self.CLR_C.append(vc.y)
                self.CLR_C.append(vc.z)
                idx += 1
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def Draw(self):
        if(self.bHide == False):  
            global g_region
            global g_rv3d
            
            global g_bMerge
            
            # Affichage du curseur
            bgl.glEnable(bgl.GL_BLEND)
            bgl.glLineWidth(1)
            if(self.bBrushSize):
                bgl.glColor4f(1.0, 0.0, 0.0, 1.0)
            else:
                bgl.glColor4f(0.97, 0.01, 0.04, 1.0)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            idx = 0
            for i in range(int(len(self.CLR_C)/3)):
                vector3d = (self.CLR_C[idx * 3] * self.BRadius + self.CurLoc.x, self.CLR_C[idx * 3  + 1] * self.BRadius + self.CurLoc.y, self.CLR_C[idx * 3 + 2] * self.BRadius + self.CurLoc.z)
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
                idx += 1
            bgl.glEnd()

            if(g_bMerge):
                bgl.glColor4f(0.0, 0.195, 1.0, 1.0)
            else:
                bgl.glColor4f(0.87, 0.28, 0.098, 1.0)

            bgl.glLineWidth(1)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            idx = 0
            for i in range(int(len(self.CLR_C)/3)):
                vector3d = (self.CLR_C[idx * 3] * 0.5 + self.CurLoc.x, self.CLR_C[idx * 3  + 1] * 0.5 + self.CurLoc.y, self.CLR_C[idx * 3 + 2] * 0.5 + self.CurLoc.z)
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                bgl.glVertex2f(*vector2d)
                idx += 1
            bgl.glEnd()

            bgl.glColor4f(0.87, 0.28, 0.098, 1.0)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            idx = 0
            for i in range(int(len(self.CLR_C)/3)):
                vector3d = (self.CLR_C[idx * 3] * self.BSize * 0.05 + self.CurLoc.x, self.CLR_C[idx * 3  + 1] * self.BSize * 0.05 + self.CurLoc.y, self.CLR_C[idx * 3 + 2] * self.BSize * 0.05 + self.CurLoc.z)
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                bgl.glVertex2f(*vector2d)
                idx += 1
            bgl.glEnd()

            if(self.bCutSize):
                font_id = 0
                blf.size(font_id, 20, 35)
                bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
                vector3d = (self.CurLoc.x, self.CurLoc.y, self.CurLoc.z)
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                dim = blf.dimensions(font_id, str(int(self.BSize)))
                blf.position(font_id, vector2d.x - dim[0]/2.0, vector2d.y - dim[1]/2.0, 0)
                blf.draw(font_id, str(int(self.BSize)))
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def DrawIcon(self, ShowCut = False, CT = False):
        if(self.bHide == False):    
            global g_region
            global g_rv3d
            global g_Cursor
            global g_rObject
            
            bgl.glEnable(bgl.GL_BLEND)
            bgl.glPointSize(10)
            bgl.glColor4f(1.0, 1.0, 1.0, 1.0)
            bgl.glBegin(bgl.GL_POINTS)
            vector3d = (self.CurLoc.x, self.CurLoc.y, self.CurLoc.z)
            vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
            if(vector2d != None):
                bgl.glVertex2f(*vector2d)
            bgl.glEnd()

#            if(ShowCut):
#                font_id = 0
#                blf.size(font_id, 25, 35)
#                bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
#                vector3d = (self.CurLoc.x, self.CurLoc.y, self.CurLoc.z)
#                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
#                dim = blf.dimensions(font_id, str(int(g_Cursor.BSize)))
#                blf.position(font_id, vector2d.x - dim[0]/2.0, vector2d.y + dim[1]/2.0, 0)
#                blf.draw(font_id, str(int(g_Cursor.BSize)))
#
#            if(CT):
#                font_id = 0
#                blf.size(font_id, 25, 35)
#                bgl.glColor4f(1.0, 0.0, 0.0, 1.0)
#                vector3d = (self.CurLoc.x, self.CurLoc.y, self.CurLoc.z)
#                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
#                dim = blf.dimensions(font_id, str(int(g_rObject.CTNCut)))
#                blf.position(font_id, vector2d.x - dim[0]/2.0, vector2d.y + dim[1]/2.0, 0)
#                blf.draw(font_id, str(int(g_rObject.CTNCut)))
#
#########################################################################################################


#########################################################################################################
class cCurvePoint:
    
    def __init__(self, _hit = None, _normal = None, _p2d = None, _bPtFromMouse = False):
        self.hit = Vector()
        if(_hit != None):
            self.hit = _hit
        self.normal = Vector()
        if(_normal != None):
            self.normal = _normal
        self.dir = Vector()
        self.persp = Vector()
        self.p2d = [-1, -1]
        if(_p2d != None):
            self.p2d[0] = _p2d[0]
            self.p2d[1] = _p2d[1]
        bPtFromMouse = _bPtFromMouse            
    #---------------------------------------------------------------------------------------------------
        
        
    #---------------------------------------------------------------------------------------------------
    def Set(self, _hit, _normal):
        self.hit = _hit
        self.normal = _normal        
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Update(self, v2d):
        self.p2d = v2d
#########################################################################################################


#########################################################################################################
class cCurve:
    
    def __init__(self):
        self.Curve = []
        self.BackCurve = []
        self.FrontCurve = []
        
        self.NbPoints = 0
        self.NbBackPoints = 0
        self.NbFrontPoints = 0
        
        self.length = -1.0

        self.MinDist = 0.1 * 0.0001
        
        self.EndPoint = -1
        
        self.Ref = []
        self.RefNbPoints = -1

        self.lastHit = Vector()
    #---------------------------------------------------------------------------------------------------
        
    
    #---------------------------------------------------------------------------------------------------
    def Reset(self):
        self.Curve.clear()
        self.BackCurve.clear()
        self.FrontCurve.clear()
        self.Ref.clear()
        
        self.EndPoint = -1
        
        self.NbPoints = 0
        self.NbBackPoints = 0
        self.NbFrontPoints = 0
        self.RefNbPoints = 0
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def AddPoint(self, _hit, _normal, _p2d = None, _bPtFromMouse = False):
        self.Curve.append(cCurvePoint(_hit, _normal, _p2d, _bPtFromMouse))
        self.NbPoints += 1
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def AddFrontPoint(self, _hit, _normal, _p2d = None, _bPtFromMouse = False):
        self.FrontCurve.append(cCurvePoint(_hit, _normal, _p2d))
        self.NbFrontPoints += 1
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def AddBackPoint(self, _hit, _normal, _p2d = None, _bPtFromMouse = False):
        self.BackCurve.append(cCurvePoint(_hit, _normal, _p2d))
        self.NbBackPoints += 1
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def Homogenize(self, cutDist = 0.001):
        global g_bCurveCurrent
        
        if(self.NbPoints > 0):
            self.length = 0
            self.Ref.clear()
            self.Ref.append(self.Curve[0])
            lenPath = len(self.Curve)
            if(self.EndPoint > 0):
                lenPath = self.EndPoint
                
            for i in range (1, lenPath):
                v = self.Curve[i].hit - self.Curve[i - 1].hit
                lc = v.length
                self.length += lc
                # Teste si la longueur est supérieure à la distance minimale
                if(lc > cutDist):
                    NnPoints = int(lc/cutDist)
                    vDir = v/NnPoints
                    for c in range(0, NnPoints):
                        np = self.Curve[i - 1].hit + vDir * c
                        
                        self.Ref.append(cCurvePoint(np, self.Curve[i - 1].normal))
                        self.RefNbPoints += 1
                else:                        
                    self.Ref.append(cCurvePoint(self.Curve[i - 1].hit, self.Curve[i - 1].normal))
                    self.RefNbPoints += 1
                        
            g_bCurveCurrent = True 
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Draw(self):
        global g_region
        global g_rv3d
        global g_bDrawing
        
        if((g_bDrawing == False) and (g_bBSurface == False) and (g_bCTRetopo == False)):
            return

        if(self.NbPoints > 0):
            # Affichage de la ligne
            bgl.glEnable(bgl.GL_BLEND)
            bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
            bgl.glLineWidth(2)
            bgl.glEnable(bgl.GL_LINE_STIPPLE)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            for CPoint in self.Curve:
                # Récupération des points de contact de la souris
                vector3d = CPoint.hit
                # Conversion en esapce 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Sauvegarde pour une prochaine utilisation
                CPoint.Update(vector2d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
            bgl.glEnd()
            bgl.glDisable(bgl.GL_LINE_STIPPLE)
            bgl.glDisable(bgl.GL_BLEND)

            # Affichage du premier point de la courbe
            bgl.glPointSize(6)
            bgl.glBegin(bgl.GL_POINTS)
            bgl.glColor4f(1.0, 0.8, 0.0, 1.0)
            bgl.glVertex2f(self.Curve[0].p2d[0], self.Curve[0].p2d[1])
            # Affichage du dernier point de la courbe
            bgl.glVertex2f(self.Curve[self.NbPoints - 1].p2d[0], self.Curve[self.NbPoints - 1].p2d[1])
            bgl.glEnd()


        if(len(self.Ref) > 0):
            # Affichage de la ligne
            bgl.glEnable(bgl.GL_BLEND)
            bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
            bgl.glLineWidth(2)
            bgl.glEnable(bgl.GL_LINE_STIPPLE)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            for CPoint in self.Ref:
                # Récupération des points de contact de la souris
                vector3d = CPoint.hit
                # Conversion en esapce 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Sauvegarde pour une prochaine utilisation
                CPoint.Update(vector2d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
            bgl.glEnd()
            bgl.glDisable(bgl.GL_LINE_STIPPLE)
            bgl.glDisable(bgl.GL_BLEND)
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Get2d(self, Index):
        return Vector((self.Curve[Index].p2d[0], self.Curve[Index].p2d[1], 0.0))
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Set(self, Index, position):
        self.Curve[Index].hit = position
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Get(self, Index):
        return self.Curve[Index].hit
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Update(self, _context = None, _callsl = None, _bPtFromMouse = False, _CTRetopo = False, _mc = None):
        global g_hit
        global g_normal
        global g_mouse_x
        global g_mouse_y
        
        lgthSq = (g_hit - self.lastHit).length_squared
                            
        # Teste la distance entre 2 points (> MinDist)
        if(g_bCTRetopo == False):
            if(lgthSq > self.MinDist):
                # Récupération du Hit point
                self.AddPoint(g_hit, g_normal, (g_mouse_x, g_mouse_y), _bPtFromMouse)
                self.lastHit = g_hit
                
        if(g_bCTRetopo):
            if(_CTRetopo == True):
                self.AddFrontPoint(g_hit, g_normal, (_mc.x, _mc.y))
                hit, normal, face = Picking(_context, _callsl, _ray_origin = g_hit, _CTRetopo = True, _fp = _mc)
                if(hit != None):
                    self.AddBackPoint(hit, normal)
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def CTUpdate(self):
        if(g_bCTRetopo):
            self.Curve.clear()
            self.Curve = self.FrontCurve
            self.NbPoints = self.NbFrontPoints
            
            self.BackCurve.reverse()
            self.Curve += self.BackCurve
            self.NbPoints += self.NbBackPoints
            self.NbBackPoints = 0
            self.NbFrontPoints = 0
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def Debug(self):
        print("----------------------------------------------------------")
        print("------------------------ Curve ---------------------------")
        for i, cp in enumerate(self.Curve):
            print(cp.hit, cp.normal, "- 2D : ", cp.p2d[0], cp.p2d[1])
        print("-------- NbPoints : ", self.NbPoints)
        print("----------------------------------------------------------")
        print("---------------------- Ref Curve -------------------------")
        for i, cp in enumerate(self.Ref):
            print(cp.hit, cp.normal)
        print("-------- NbPoints (Ref): ", len(self.Ref))
        print("-------- length : ", self.length)
        print("----------------------------------------------------------")
        print("")

#########################################################################################################



##################################################################################################
class cVertex:
    
    def __init__(self, x, y, z, faceIdx = None, select = False, copy = None, BSelect = False):
        self.x = x
        self.y = y
        self.z = z
        self.select = select
        self.BS_select = BSelect
        self.numFaces = 0
        self.x2d = -1
        self.y2d = -1
        self.faceIndex = []
        if(faceIdx != None):
            self.faceIndex = []
            for i, n in enumerate(faceIdx):
                self.faceIndex.append(n)                            
                self.numFaces += 1
    #---------------------------------------------------------------------------------------------------
                
                
    #---------------------------------------------------------------------------------------------------
    def SetCoord(self, pos):
        self.x = pos.x
        self.y = pos.y
        self.z = pos.z
    #---------------------------------------------------------------------------------------------------
                
                
    #---------------------------------------------------------------------------------------------------
    def Update2d(self, pos2d):
        self.x2d, self.y2d = pos2d
    #---------------------------------------------------------------------------------------------------
                
                
    #---------------------------------------------------------------------------------------------------
    def AddFace(self, index):
        if(isinstance(index, int)):
            bReplace = False
            for i in range(len(self.faceIndex[0:])):
                if(self.faceIndex[i] < 0):
                    self.faceIndex[i] = index
                    self.numFaces += 1
                    bReplace = True
                    break
            if(bReplace == False):                
                self.faceIndex.append(index)
                self.numFaces += 1
            
        if(isinstance(index, tuple)):
            n = 0
            for i in range(len(self.faceIndex[0:])):
                if(self.faceIndex[i] < 0):
                    self.faceIndex[i] = index[n]
                    n += 1
                    self.numFaces += 1
            for i in range(len(index) - n):
                self.faceIndex.append(index[i + n])
                self.numFaces += 1
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def BS_Select(self, value):
        self.BS_select = value
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def Select(self, value):
        self.select = value
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def Selected(self):
        return self.select
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def Get(self):
        return (Vector((self.x, self.y, self.z)))
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def Get2d(self):
        return ((self.x2d, self.y2d))
    #---------------------------------------------------------------------------------------------------

       
    #---------------------------------------------------------------------------------------------------
    def Get2dv(self):
        return Vector((self.x2d, self.y2d, 0.0))
       
##################################################################################################

                

##################################################################################################
class cVertices:
    
    def __init__(self):
        self.list = []
        self.TotalVertices = 0
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def AddVertex(self, vertex):
        if(isinstance(vertex, cVertex)):
            self.list.append(vertex)
            self.TotalVertices  += 1
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Move(self, VertexLst, Location):
        for v in VertexLst:
            self.list[v].SetCoord(Location)
    #---------------------------------------------------------------------------------------------------

            
    #---------------------------------------------------------------------------------------------------
    def GetFaceIndex(self, Index):
        try:
            return self.list[Index].faceIndex
        except:
            print("Error in GetFaceIndex() : ", Index)
    #---------------------------------------------------------------------------------------------------

            
    #---------------------------------------------------------------------------------------------------
    def Get(self, Index):
        return Vector((self.list[Index].x, self.list[Index].y, self.list[Index].z))
    #---------------------------------------------------------------------------------------------------

            
    #---------------------------------------------------------------------------------------------------
    def Get2d(self, Index):
        try:
            return Vector((self.list[Index].x2d, self.list[Index].y2d, 0.0))
        except:
            print("Error in Get2d() : ", Index)
    #---------------------------------------------------------------------------------------------------

            
    #---------------------------------------------------------------------------------------------------
    def Update(self, IndexFace, Index):
        self.list[Index].AddFace(IndexFace)
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Debug(self):
        print("-------------------- Total vertices --> ", self.TotalVertices)
        print("-------------------------------------------------------------")
        for i, v in enumerate(self.list):
#            print(i, "- (", v.x, ",", v.y, ",", v.z, ") - Nb Faces :", v.numFaces, "- Faces Index :", v.faceIndex[0:])
            print(i, "- Nb Faces :", v.numFaces, "- Faces Index :", v.faceIndex[0:])
##################################################################################################
            


##################################################################################################
class cFace:
    
    def __init__(self, vertIdx = None, center = None, cx2d = -1, cy2d = -1, select = False):
        # Valeurs par défaut
        self.VertexIdx = []
        self.NbVertices = 0
        
        self.Normal = Vector()
        
        if(isinstance(vertIdx, tuple)):
            for i in range(len(vertIdx)):
                self.VertexIdx.append(vertIdx[i])
                self.NbVertices += 1

        self.CalculateNormal()
                
        self.center = center
            
        self.cx2d = cx2d
        self.cy2d = cy2d
        
        self.select = select
        
        self.ps = 0
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def CalculateNormal(self):
        global g_rObject
        
        if(len(self.VertexIdx) > 1):
            edge0 = g_rObject.Vertices.Get(self.VertexIdx[0]) - g_rObject.Vertices.Get(self.VertexIdx[1])
            edge1 = g_rObject.Vertices.Get(self.VertexIdx[1]) - g_rObject.Vertices.Get(self.VertexIdx[2])
            self.Normal = edge0.cross(edge1)
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def Inverse(self):
        if(self.NbVertices == 3):
            tmp = self.VertexIdx[0]
            self.VertexIdx[0] = self.VertexIdx[2]
            self.VertexIdx[2] = tmp
        else:
            tmp = self.VertexIdx[0]
            self.VertexIdx[0] = self.VertexIdx[3]
            self.VertexIdx[3] = tmp
            tmp = self.VertexIdx[1]
            self.VertexIdx[1] = self.VertexIdx[2]
            self.VertexIdx[2] = tmp
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def GetCenter(self):
        return self.center
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def ReplaceVertex(self, i0, i1):
        for i, v in enumerate(self.VertexIdx):
            if(v == i0):
                self.VertexIdx[i] = i1
                break
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def Select(self, value):
        self.select = value
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def SetCenter2d(self, Center2d):
        self.cx2d, self.cy2d = Center2d
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def AddVertex(self, index):
        if(isinstance(index, int)):
            bReplace = False
            for i in range(len(self.VertexIdx[0:])):
                if(self.VertexIdx[i] < 0):
                    self.VertexIdx[i] = index
                    self.NbVertices += 1
                    bReplace = True
                    break
            if(bReplace == False):                
                self.VertexIdx.append(index)
                self.NbVertices += 1
            
        if(isinstance(index, tuple)):
            n = 0
            for i in range(len(self.VertexIdx[0:])):
                if(self.VertexIdx[i] < 0):
                    self.VertexIdx[i] = index[n]
                    n += 1
                    self.NbVertices += 1
            for i in range(len(index) - n):
                self.VertexIdx.append(index[i + n])
                self.NbVertices += 1
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def GetVerticesIndex(self):
        return self.VertexIdx
    #---------------------------------------------------------------------------------------------------
##################################################################################################


        
##################################################################################################
class cFaces:
    
    def __init__(self):
        self.list = []
        self.TotalFaces = 0
        self.DelFacesList = []
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def AddFace(self, face):
        self.list.append(face)
        self.TotalFaces += 1
        return self.TotalFaces - 1
    #---------------------------------------------------------------------------------------------------
        
        
    #---------------------------------------------------------------------------------------------------
    def CenterUpdate(self, nface, center):
        self.list[nface].center = center
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def GetVerticesIndex(self, nface):
        try:
            return self.list[nface].VertexIdx
        except:
            print("Error : GetVerticesIndex : ", nface, " - TotalFaces : ", self.TotalFaces)
            
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def ReplaceVertex(self, index, i0, i1):
        self.list[index].ReplaceVertex(i0, i1)
        
    #---------------------------------------------------------------------------------------------------
            

    #---------------------------------------------------------------------------------------------------
    def Debug(self):
        print("-------------------------------------------------------------")
        print("-------------------- Total Faces --> ", self.TotalFaces)
        print("-------------------------------------------------------------")
        for i, f in enumerate(self.list):
            print(i, "- Nb Points :", f.NbVertices, " - ", f.VertexIdx)
##################################################################################################


        
##################################################################################################
class cObject:
    
    def __init__(self):
        self.Vertices = cVertices()
        self.Faces = cFaces()
        self.pI0 = self.pI1 = self.pI0End = self.pI1End = -1
        self.FaceStart = self.FaceEnd = -1
        self.GeomPath = cCurve()
        self.GeomBSPath = cCurve()

        self.AM_dist = 0.05
        
        self.NbEdges = 0
        
        self.SelectPointDist = 10
        self.SaveSel = []
 
        self.UndoVL = []
        self.UndoFL = []
        self.UndoNE = []
        
        self.PointIdx = []
        self.MergeIdx = -1
        self.SelectedFace = -1

        self.CTNCut = 10
        self.CTSel = []
        self.fpCT = None
        
        self.Sg_hit = None
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def ResetCollapse(self):
        global g_bCurveCurrent
        global g_PosSizeIcon
        global g_PosCutIcon
        global g_PosCutBef_in
        global g_PosCutBef_out

        global g_PosCutCTIcon
        global g_CTCurve
        global g_CTNCurve
        
        self.pI0 = self.pI1 = self.pI0End = self.pI1End = -1
        self.FaceStart = self.FaceEnd = -1
        self.CTSel.clear()
        g_bCurveCurrent = False
        g_PosSizeIcon = g_PosCutIcon = g_PosCutBef_in =  g_PosCutBef_out = None
    #---------------------------------------------------------------------------------------------------

        
    #---------------------------------------------------------------------------------------------------
    def Debug(self):
        print("------------------------ Object debug -----------------------")
        self.Vertices.Debug()
        self.Faces.Debug()
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def AddFace(self, face):
        n = self.Faces.AddFace(face)
        # mise à jour des vertices
        for i, nf in enumerate(face.VertexIdx):
            self.Vertices.Update(n, nf)
    #---------------------------------------------------------------------------------------------------
        

    #---------------------------------------------------------------------------------------------------
    def AddVertex(self, vertex):
        self.Vertices.AddVertex(vertex)        
    #---------------------------------------------------------------------------------------------------

                
    #---------------------------------------------------------------------------------------------------
    def Merge(self, IndexFrom, IndexTo):
        # Update vertex position
        try:
            self.Vertices.list[IndexTo].SetCoord(self.Vertices.list[IndexFrom].Get())
        except:
            print("Error in SetCoord() : ", IndexTo, IndexFrom)
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def CreateGeometry(self):
        global g_bCurveCurrent
        global g_PosCutIcon
        global g_PosSizeIcon

        global g_SizeIcon
        global g_CutIcon
        
        global g_bAutoMerge
        
        global g_rv3d

        
        if((g_Cursor == None) or (g_Curve == None) or (g_bCurveCurrent == False)):
            return
        
        # Maintenant la réference de la courbe, c'est 'RefCurve' pour travailler dessus
        NbPoints = g_Curve.length/g_Cursor.BSize
        if((NbPoints == 0) or (g_Curve.NbPoints < 2)):
           return
        DistPoint = g_Curve.length/NbPoints
        DistPoint_squared = DistPoint * DistPoint
        
        RefPoint = 0
        self.GeomPath.Reset()
        self.GeomPath.AddPoint(g_Curve.Ref[0].hit, g_Curve.Ref[0].normal)
        
        LastIndx = g_Curve.NbPoints - 1
        if(LastIndx <= 1):
            return 
        
        lp = 0
        ldist = g_Curve.length/g_Cursor.BSize
        for i in range (1, g_Curve.RefNbPoints):
            v1 = g_Curve.Ref[i].hit
            v0 = g_Curve.Ref[i - 1].hit
            v = v1 - v0
            lp += v.length
            if(lp >= ldist):
                lp = 0
                self.GeomPath.AddPoint(g_Curve.Ref[i].hit, g_Curve.Ref[i].normal)

        self.GeomPath.AddPoint(g_Curve.Ref[len(g_Curve.Ref) - 1].hit, g_Curve.Ref[len(g_Curve.Ref) - 1].normal)
                
        for i in range(1, self.GeomPath.NbPoints):
            # Calcul de la direction, si la liste n'est pas vide
            PrevIdx = i - 1
            PrevPerps = Vector((0.0, 0.0, 0.0))
            if(PrevIdx >= 0):
                Hit = self.GeomPath.Curve[i].hit
                PrevHit = self.GeomPath.Curve[PrevIdx].hit
                PrevNormal = self.GeomPath.Curve[PrevIdx].normal
                PrevDir = Hit - PrevHit
                PrevDir = PrevDir.normalized()
                
                self.GeomPath.Curve[PrevIdx].dir = PrevDir
                # Calcul du vecteur perpendiculaire
                PrevPerps = PrevDir.cross(PrevNormal)
                # Sauvegarde de la perpendiculaire
                self.GeomPath.Curve[PrevIdx].persp = PrevPerps
                
                self.GeomPath.Curve[i].persp = PrevPerps

        BRad = g_Cursor.BRadius
        incBRad = 0.0
        if(self.pI0 >= 0):
            # Calcul du BRadius
            edge = self.Vertices.Get(self.pI1) - self.Vertices.Get(self.pI0)
            BRad = edge.length * 10
        if(self.pI0End >= 0):
            edge = self.Vertices.Get(self.pI1End) - self.Vertices.Get(self.pI0End)
            BRadEnd = edge.length * 10
            incBRad = (BRadEnd - BRad)/g_Cursor.BSize
            incBRad /= 20.0
                
        origin = (0.0, 0.0, 0.0)
        self.Facescale = BRad/20.0
        NbPointsCreated = 0
        
        nVertex = []
        MergeList = []
        nVn = self.Vertices.TotalVertices
        if(self.GeomPath.NbPoints > 1):
            # Calcul des points
            for idx in range(0, self.GeomPath.NbPoints):
                hit = self.GeomPath.Curve[idx].hit
                Persp = self.GeomPath.Curve[idx].persp * self.Facescale
                self.Facescale += incBRad
                pt0 = hit + Persp
                pt1 = hit - Persp
                if(self.pI0 < 0):
                    self.AddVertex(cVertex(pt0.x, pt0.y, pt0.z))
                    nVertex.append(nVn)
                    nVn += 1
                    self.AddVertex(cVertex(pt1.x, pt1.y, pt1.z))
                    nVertex.append(nVn)
                    nVn += 1
                    NbPointsCreated += 2
                else:
                    if(idx > 0):
                        if(self.pI0End < 0):
                            self.AddVertex(cVertex(pt0.x, pt0.y, pt0.z))
                            nVertex.append(nVn)
                            nVn += 1
                            self.AddVertex(cVertex(pt1.x, pt1.y, pt1.z))
                            nVertex.append(nVn)
                            nVn += 1
                            NbPointsCreated += 2
                        else:
                            if(idx < self.GeomPath.NbPoints - 1):
                                self.AddVertex(cVertex(pt0.x, pt0.y, pt0.z))
                                nVertex.append(nVn)
                                nVn += 1
                                self.AddVertex(cVertex(pt1.x, pt1.y, pt1.z))
                                nVertex.append(nVn)
                                nVn += 1
                                NbPointsCreated += 2
            
            g_CutIcon.RotAndPos(hit, self.GeomPath.Curve[self.GeomPath.NbPoints - 1].normal)
            g_PosCutIcon = hit
            if(self.pI0< 0):
                g_SizeIcon.RotAndPos(pt0, self.GeomPath.Curve[self.GeomPath.NbPoints - 1].normal)
                g_PosSizeIcon = pt0
            else:
                g_PosSizeIcon = None

            # Création des faces
            nFaceIdx = self.NbEdges
            CurrentFace = self.Faces.TotalFaces
            
            look_at, camera_pos = camera(bpy.context.space_data.region_3d)

            if((g_Cursor.BSize == 1) and (self.pI0End >= 0)):
                # On créé la face directement
                middle = self.Vertices.Get(self.pI0) + self.Vertices.Get(self.pI1) + self.Vertices.Get(self.pI1End) + self.Vertices.Get(self.pI0End)/4.0
                fc = cFace((self.pI0, self.pI1, self.pI0End, self.pI1End), middle)
                vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                ps = vs.dot(fc.Normal)
                if(not g_rv3d.is_perspective):
                    ps = -ps
                if(ps > 0):
                    fc.Inverse()
                    fc.CalculateNormal()
                self.AddFace(fc)
                self.NbEdges += NbPointsCreated
            else:                
                for idx in range(0, self.GeomPath.NbPoints - 1):
                    # Calcul le centre de la face
                    hit0 = self.GeomPath.Curve[idx].hit
                    hit1 = self.GeomPath.Curve[idx + 1].hit
                    Center = hit0 + ((hit1 - hit0) / 2.0)
                    if(self.pI0 < 0):
                        fc = cFace((nFaceIdx + idx * 2, nFaceIdx + idx * 2 + 1, nFaceIdx + (idx + 1) * 2 + 1, nFaceIdx + (idx + 1) * 2), Center)
                        vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                        ps = vs.dot(fc.Normal)
                        if(not g_rv3d.is_perspective):
                            ps = -ps
                        if(ps > 0):
                            fc.Inverse()
                            fc.CalculateNormal()
                        self.AddFace(fc)
                    else:
                        if(idx > 0):
                            if(self.FaceEnd >= 0):
                                if(idx == self.GeomPath.NbPoints - 2):
                                    fc = cFace((nFaceIdx + (idx - 1)  * 2, nFaceIdx + (idx - 1) * 2 + 1, self.pI1End, self.pI0End), Center)
                                    vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                                    ps = vs.dot(fc.Normal)
                                    if(not g_rv3d.is_perspective):
                                        ps = -ps
                                    if(ps > 0):
                                        fc.Inverse()
                                        fc.CalculateNormal()
                                    self.AddFace(fc)
                                else:
                                    fc = cFace((nFaceIdx + (idx - 1)  * 2, nFaceIdx + (idx - 1) * 2 + 1, nFaceIdx + ((idx - 1) + 1) * 2 + 1, nFaceIdx + ((idx - 1) + 1) * 2), Center)
                                    vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                                    ps = vs.dot(fc.Normal)
                                    if(not g_rv3d.is_perspective):
                                        ps = -ps
                                    if(ps > 0):
                                        fc.Inverse()
                                        fc.CalculateNormal()
                                    self.AddFace(fc)
                            else:
                                fc = cFace((nFaceIdx + (idx - 1)  * 2, nFaceIdx + (idx - 1) * 2 + 1, nFaceIdx + ((idx - 1) + 1) * 2 + 1, nFaceIdx + ((idx - 1) + 1) * 2), Center)
                                vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                                ps = vs.dot(fc.Normal)
                                if(not g_rv3d.is_perspective):
                                    ps = -ps
                                if(ps > 0):
                                    fc.Inverse()
                                    fc.CalculateNormal()
                                self.AddFace(fc)
                        else:
                            fc = cFace((self.pI1, self.pI0, nFaceIdx + ((idx - 1) + 1) * 2 + 1, nFaceIdx + ((idx - 1) + 1) * 2), Center)
                            vs = camera_pos - self.Vertices.Get(fc.VertexIdx[0])
                            ps = vs.dot(fc.Normal)
                            if(not g_rv3d.is_perspective):
                                ps = -ps
                            if(ps > 0):
                                fc.Inverse()
                                fc.CalculateNormal()
                            self.AddFace(fc)
                self.NbEdges += NbPointsCreated
                
        # Test if vertices to merge
        Merged = []
        if(g_bAutoMerge):
            for i, nv in enumerate(nVertex):
                min = None
                sj = -1
                for j, v in enumerate(self.Vertices.list):
                    if((j not in nVertex) and (j not in Merged)):
                        vec = self.Vertices.Get(nv) - v.Get()
                        if(vec.length <= self.AM_dist):
                            if(min == None):
                                min = vec.length
                                sj = j
                            else:
                                if(vec.length < min):
                                    min = vec.length
                                    sj = j
                if(sj != -1):
                    MergeList.append([sj, nv])
                    Merged.append(sj)
    
            for i in range(len(MergeList)):
                if((MergeList[i][0] != None) and (MergeList[i][1] != None)):
                    self.Merge(MergeList[i][0], MergeList[i][1])
                    
                    
            if(len(nVertex) >= 4):                    
                vStart0 = nVertex[0]
                vStart1 = nVertex[1]
                vEnd0   = nVertex[len(nVertex) - 2]
                vEnd1   = nVertex[len(nVertex) - 1]
                vec = self.Vertices.Get(vStart0) - self.Vertices.Get(vEnd0)
                if(vec.length <= self.AM_dist):
                    self.Merge(vStart0, vEnd0)
                vec = self.Vertices.Get(vStart1) - self.Vertices.Get(vEnd1)
                if(vec.length <= self.AM_dist):
                    self.Merge(vStart1, vEnd1)
    #---------------------------------------------------------------------------------------------------
                

    #---------------------------------------------------------------------------------------------------
    def CreateBSGeometry(self, nc):
        global g_Selection
        global g_BSCurve
        
        if(g_BSCurrent < 0):
            return

        if(len(g_Selection) > 0):
            NbPoints = g_BSCurve[nc].length/(len(g_Selection) - 1)
            if((NbPoints == 0) or (g_BSCurve[nc].NbPoints < 2)):
               return
            DistPoint = g_BSCurve[nc].length/NbPoints
            DistPoint_squared = DistPoint * DistPoint
            
            RefPoint = 0
            self.GeomBSPath.Reset()
            self.GeomBSPath.AddPoint(g_BSCurve[nc].Ref[0].hit, g_BSCurve[nc].Ref[0].normal)
            
            LastIndx = g_BSCurve[nc].NbPoints - 1
            if(LastIndx <= 1):
                return 
            
            lp = 0
            ldist = g_BSCurve[nc].length/(len(g_Selection) - 1)
            for i in range (1, g_BSCurve[nc].RefNbPoints):
                v1 = g_BSCurve[nc].Ref[i].hit
                v0 = g_BSCurve[nc].Ref[i - 1].hit
                v = v1 - v0
                lp += v.length
                if(lp >= ldist):
                    lp = 0
                    self.GeomBSPath.AddPoint(g_BSCurve[nc].Ref[i].hit, g_BSCurve[nc].Ref[i].normal)

            self.GeomBSPath.AddPoint(g_BSCurve[nc].Ref[len(g_BSCurve[nc].Ref) - 1].hit, g_BSCurve[nc].Ref[len(g_BSCurve[nc].Ref) - 1].normal)
                    
            NbPointsCreated = 0
            nvtx = self.Vertices.TotalVertices
            nSel = []
            
            nVertex = []
            MergeList = []
            nVn = self.Vertices.TotalVertices
            if(self.GeomBSPath.NbPoints > 1):
                # Calcul des points
                for idx in range(0, self.GeomBSPath.NbPoints):
                    hit = self.GeomBSPath.Curve[idx].hit
                    self.AddVertex(cVertex(hit.x, hit.y, hit.z))
                    nSel.append(nvtx)
                    NbPointsCreated += 1
                    nvtx += 1
                    nVertex.append(nVn)
                    nVn += 1
                    
            # Creation des faces
            ps = 0.0
            for vi in range(len(nSel) - 1):
                center = self.Vertices.Get(g_Selection[vi]) + self.Vertices.Get(g_Selection[vi + 1]) + self.Vertices.Get(nSel[vi + 1]) + self.Vertices.Get(nSel[vi])/4.0
                fc = cFace((g_Selection[vi], g_Selection[vi + 1], nSel[vi + 1], nSel[vi]), center)
                if(ps == 0.0):
                    vs = g_BSVs[nc] - self.Vertices.Get(fc.VertexIdx[0])
                    ps = vs.dot(fc.Normal)
                    if(not g_rv3d.is_perspective):
                        ps = -ps

                if(ps > 0):
                    fc.Inverse()
                    fc.CalculateNormal()
                self.AddFace(fc)
                
            g_Selection = nSel.copy()                
            self.NbEdges += NbPointsCreated
            
            # On teste s'il y a des points à merger
            if(g_bAutoMerge):
                for i, nv in enumerate(nVertex):
                    min = None
                    sj = -1
                    for j, v in enumerate(self.Vertices.list):
                        if(j not in nVertex):
                            vec = self.Vertices.Get(nv) - v.Get()
                            if(vec.length <= self.AM_dist/1.0):
                                if(min == None):
                                    min = vec.length
                                    sj = j
                                else:
                                    if(vec.length < min):
                                        min = vec.length
                                        sj = j
                    if(sj != -1):
                        MergeList.append([sj, nv])
        
                for i in range(len(MergeList)):
                    self.Merge(MergeList[i][0], MergeList[i][1])
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def CreateCTGeometry(self, nc):
        global g_CTCurve
        global g_CTNCurve
        global g_PosCutCTIcon
        global g_FirstPoint
        global g_EndPoint
        
        if(g_CTNCurve < 0):
            return
        
        NbPoints = g_CTCurve[nc].length/self.CTNCut
        if((NbPoints == 0) or (g_CTCurve[nc].NbPoints < 2)):
            return
        DistPoint = g_CTCurve[nc].length/NbPoints
        DistPoint_squared = DistPoint * DistPoint
        
        RefPoint = 0
        self.GeomBSPath.Reset()
        self.GeomBSPath.AddPoint(g_CTCurve[nc].Ref[0].hit, g_CTCurve[nc].Ref[0].normal)
        
        LastIndx = g_CTCurve[nc].NbPoints - 1
        if(LastIndx <= 1):
            return 
        
        lp = 0
        ldist = g_CTCurve[nc].length/self.CTNCut
        for i in range (1, g_CTCurve[nc].RefNbPoints):
            v1 = g_CTCurve[nc].Ref[i].hit
            v0 = g_CTCurve[nc].Ref[i - 1].hit
            v = v1 - v0
            lp += v.length
            if(lp >= ldist):
                lp = 0
                self.GeomBSPath.AddPoint(g_CTCurve[nc].Ref[i].hit, g_CTCurve[nc].Ref[i].normal)

        self.GeomBSPath.AddPoint(g_CTCurve[nc].Ref[len(g_CTCurve[nc].Ref) - 1].hit, g_CTCurve[nc].Ref[len(g_CTCurve[nc].Ref) - 1].normal)
                
        NbPointsCreated = 0
        nvtx = self.Vertices.TotalVertices
        nSel = []
        if(self.GeomBSPath.NbPoints > 1):
            # Calcul des points
            for idx in range(0, self.GeomBSPath.NbPoints):
                hit = self.GeomBSPath.Curve[idx].hit
                self.AddVertex(cVertex(hit.x, hit.y, hit.z))
                nSel.append(nvtx)
                NbPointsCreated += 1
                nvtx += 1

        if(nc == 0):
            g_PosCutCTIcon = self.GeomBSPath.Curve[0].hit.copy()
        if(nc == 1):
            if(g_PosCutCTIcon != None):
                g_PosCutCTIcon += self.GeomBSPath.Curve[0].hit
                g_PosCutCTIcon /= 2.0
                g_CutIcon.RotAndPos(g_PosCutCTIcon, self.GeomBSPath.Curve[0].normal)

        if(len(self.CTSel) > 0):
            # Creation des faces
            if(nc > 0):
                ps = 0.0
                for vi in range(len(nSel) - 1):
                    center = (self.Vertices.Get(self.CTSel[vi]) + self.Vertices.Get(self.CTSel[vi + 1]) + self.Vertices.Get(nSel[vi + 1]) + self.Vertices.Get(nSel[vi]))/4.0
                    if(g_Inverse):
                        fc = cFace((self.CTSel[vi], nSel[vi],  nSel[vi + 1], self.CTSel[vi + 1]), center)
                    else:                        
                        fc = cFace((self.CTSel[vi],  self.CTSel[vi + 1], nSel[vi + 1], nSel[vi]), center)
                    self.AddFace(fc)

        if(g_FirstPoint and g_EndPoint):
            if(nc > 0):
                if(len(nSel) > 0):
                    self.Merge(self.fpCT[0], self.fpCT[1])
                    self.Merge(nSel[0], nSel[len(nSel) - 1])

        if(len(nSel) > 0):
            self.fpCT = (nSel[0], nSel[len(nSel) - 1])
        
        self.CTSel = nSel
                
        self.NbEdges += NbPointsCreated
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def PinEndUpdate(self):
        global g_Curve
        global g_mouse_x
        global g_mouse_y
        global g_normal
        
        if(self.FaceStart < 0):
            return
        else:
            # Si FaceStart est différent de -1, on teste si on s'est pas arrété sur une face
            for i, face in enumerate(self.Faces.list):
                if(face.ps < 0.0):
                    face.Select(False)
                    if(face.NbVertices == 3):
                        VertList = face.GetVerticesIndex()
                        vert0 = self.Vertices.Get2d(VertList[0])
                        vert1 = self.Vertices.Get2d(VertList[1])
                        vert2 = self.Vertices.Get2d(VertList[2])
                        
                        ret = geometry.intersect_point_tri_2d(Vector((g_mouse_x, g_mouse_y, 0.0)), vert0, vert1, vert2)
                        if(ret !=0):
                            face.Select(True)
                            self.FaceEnd = i
                    else:
                        VertList = face.GetVerticesIndex()
                        vert0 = self.Vertices.Get2d(VertList[0])
                        vert1 = self.Vertices.Get2d(VertList[1])
                        vert2 = self.Vertices.Get2d(VertList[2])
                        vert3 = self.Vertices.Get2d(VertList[3])
                        
                        ret = geometry.intersect_point_quad_2d(Vector((g_mouse_x, g_mouse_y, 0.0)), vert0, vert1, vert2, vert3)
                        if(ret !=0):
                            face.Select(True)
                            self.FaceEnd = i

                if(self.FaceEnd >= 0):
                    # Recherche le segment qui a été coupé
                    nedge = g_Curve.NbPoints - 1
                    while nedge > 0: 
                        p1 = g_Curve.Get2d(nedge)
                        p0 = g_Curve.Get2d(nedge - 1)

                        nedge -= 1
                        # Teste tous les edges de la face par rapport au dernier edge
                        nf = 0
                        FaceEnd = self.Faces.list[self.FaceEnd]
                        ListIndex = FaceEnd.GetVerticesIndex()
            
                        Found = False
                        nf = 0
                        for i in range(FaceEnd.NbVertices - 1):
                            pCE0 = self.Vertices.Get2d(ListIndex[nf])
                            pCE1 = self.Vertices.Get2d(ListIndex[nf + 1])
                            ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                            if(ret != None):
                                self.pI0End = ListIndex[nf]
                                self.pI1End = ListIndex[nf + 1]
                                
                                # Calcul du milieu du segment
                                p0 = self.Vertices.Get(self.pI0End)
                                p1 = self.Vertices.Get(self.pI1End)
                                middle = (p0 + p1) / 2.0
                                
                                g_Curve.Set((nedge - 1), middle)
                                g_Curve.EndPoint = nedge

                                Found = True
                                
                                break
                            nf += 1

                        if(Found == False):
                            if(FaceEnd.NbVertices == 4):
                                pCE0 = self.Vertices.Get2d(ListIndex[3])
                                pCE1 = self.Vertices.Get2d(ListIndex[0])
                                ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                                if(ret != None):
                                    self.pI0End = ListIndex[3]
                                    self.pI1End = ListIndex[0]
                                    
                                    # Calcul du milieu du segment
                                    p0 = self.Vertices.Get(self.pI0End)
                                    p1 = self.Vertices.Get(self.pI1End)
                                    middle = (p0 + p1) / 2.0
                                    
                                    g_Curve.Set((nedge - 1), middle)
                                    g_Curve.EndPoint = nedge
                            else:
                                pCE0 = self.Vertices.Get2d(ListIndex[2])
                                pCE1 = self.Vertices.Get2d(ListIndex[0])
                                ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                                if(ret != None):
                                    self.pI0End = ListIndex[2]
                                    self.pI1End = ListIndex[0]
                                    
                                    # Calcul du milieu du segment
                                    p0 = self.Vertices.Get(self.pI0End)
                                    p1 = self.Vertices.Get(self.pI1End)
                                    middle = (p0 + p1) / 2.0

                                    g_Curve.Set((nedge - 1), middle)
                                    g_Curve.EndPoint = nedge
                        else:
                            break
    #---------------------------------------------------------------------------------------------------
                

    #---------------------------------------------------------------------------------------------------
    def PinUpdate(self):
        global g_Curve
        global g_mouse_x
        global g_mouse_y
        global g_normal
        
        if((self.SelectedFace < 0) or (self.pI0 >= 0)):
            return

        if(g_Curve.NbPoints > 1):
            # Recherche du point de départ 
            p1 = Vector((g_mouse_x, g_mouse_y, 0.0))
            p0 = g_Curve.Get2d(0)
            # Teste tous les edges de la face par rapport au dernier edge
            nf = 0
            FaceStart = self.Faces.list[self.FaceStart]
            ListIndex = FaceStart.GetVerticesIndex()

            for i in range(FaceStart.NbVertices - 1):
                pCE0 = self.Vertices.Get2d(ListIndex[nf])
                pCE1 = self.Vertices.Get2d(ListIndex[nf + 1])
                ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                if(ret != None):
                    self.pI0 = ListIndex[nf]
                    self.pI1 = ListIndex[nf + 1]
                    
                    # Calcul du milieu du segment
                    p0 = self.Vertices.Get(self.pI0)
                    p1 = self.Vertices.Get(self.pI1)
                    middle = (p0 + p1) / 2.0
                    g_Curve.Reset()
                    g_Curve.AddPoint(middle, g_normal)

                nf += 1
                
            if(FaceStart.NbVertices == 4):
                pCE0 = self.Vertices.Get2d(ListIndex[3])
                pCE1 = self.Vertices.Get2d(ListIndex[0])
                ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                if(ret != None):
                    self.pI0 = ListIndex[3]
                    self.pI1 = ListIndex[0]
                    
                    # Calcul du milieu du segment
                    p0 = self.Vertices.Get(self.pI0)
                    p1 = self.Vertices.Get(self.pI1)
                    middle = (p0 + p1) / 2.0
                    g_Curve.Reset()
                    g_Curve.AddPoint(middle, g_normal)
            else:
                pCE0 = self.Vertices.Get2d(ListIndex[2])
                pCE1 = self.Vertices.Get2d(ListIndex[0])
                ret = geometry.intersect_line_line_2d(p0, p1, pCE0, pCE1)
                if(ret != None):
                    self.pI0 = ListIndex[2]
                    self.pI1 = ListIndex[0]
                    
                    # Calcul du milieu du segment
                    p0 = self.Vertices.Get(self.pI0)
                    p1 = self.Vertices.Get(self.pI1)
                    middle = (p0 + p1) / 2.0
                    g_Curve.Reset()
                    g_Curve.AddPoint(middle, g_normal)
    #---------------------------------------------------------------------------------------------------



    #---------------------------------------------------------------------------------------------------
    def MergePointsInHit(self, hit):
        global g_Cursor

        if(hit != None):
            for i, vert in enumerate(self.Vertices.list):
                l = (hit - vert.Get()).length
                if(l < g_Cursor.BRadius/20.0):
                    vert.SetCoord(hit)
    #---------------------------------------------------------------------------------------------------



    #---------------------------------------------------------------------------------------------------
    def Update(self):
        global g_bPtMoving
        global g_bFaceMoving
        global g_bFaceCreating

        global g_bDrawing
        global g_bMerge
        
        global g_mouse_x
        global g_mouse_y
        global g_hit
        
        global g_bLeftBM
        global g_bRightBM
        global g_CTRL
        # Edit:
        global g_SHIFT
        g_rObject
        # Edit:

        global g_Selection
        
        if(g_bFaceCreating):
            return

        # Teste si un point est dessous du curseur
        if((g_bPtMoving == False) and (g_bFaceMoving == False)):
            if(g_CTRL == 'PRESS'):
                # Teste si un point est dessous du curseur
                self.PointIdx.clear()
                for i, vert in enumerate(self.Vertices.list):
                    vert.Select(False)
                    if((abs(g_mouse_x - vert.Get2dv().x) < self.SelectPointDist) and (abs(g_mouse_y - vert.Get2dv().y) < self.SelectPointDist)):
                        bHide = False
                        for n, nface in enumerate(vert.faceIndex):
                            if(nface >= 0):
                                if(self.Faces.list[nface].ps > 0.0):
                                    bHide = True
                                    break
                        if(bHide == False):                            
                            self.PointIdx.append(i)
                            vert.Select(True)
                            self.SelectedFace = -1
                # Merge points around g_hit
                #g_rObject.MergePointsInHit(g_hit)

            else:
                for i, v in enumerate(self.PointIdx):
                    self.Vertices.list[v].Select(False)
                self.PointIdx.clear()

            # Si aucun point de selectionné, on teste les faces
            if(len(self.PointIdx) == 0):
                self.SelectedFace = -1
                for i, face in enumerate(self.Faces.list):
                    face.Select(False)
                    if(face.ps < 0.0):
                        if(face.NbVertices == 3):
                            VertList = face.GetVerticesIndex()
                            vert0 = self.Vertices.Get2d(VertList[0])
                            vert1 = self.Vertices.Get2d(VertList[1])
                            vert2 = self.Vertices.Get2d(VertList[2])
                            
                            if(vert0 and vert1 and vert2):
                                ret = geometry.intersect_point_tri_2d(Vector((g_mouse_x, g_mouse_y, 0.0)), vert0, vert1, vert2)
                                if(ret !=0):
                                    face.Select(True)
                                    self.SelectedFace = i
                        else:
                            VertList = face.GetVerticesIndex()
                            vert0 = self.Vertices.Get2d(VertList[0])
                            vert1 = self.Vertices.Get2d(VertList[1])
                            vert2 = self.Vertices.Get2d(VertList[2])
                            vert3 = self.Vertices.Get2d(VertList[3])
                            
                            if(vert0 and vert1 and vert2 and vert3):
                                ret = geometry.intersect_point_quad_2d(Vector((g_mouse_x, g_mouse_y, 0.0)), vert0, vert1, vert2, vert3)
                                if(ret != 0):
                                    face.Select(True)
                                    self.SelectedFace = i
        else:
            # Moving point
            if(g_bPtMoving):
                if(g_hit != None):
                    self.Vertices.Move(self.PointIdx, g_hit)
                    
                    g_bMerge = False
                    min = None
                    for i, vert in enumerate(self.Vertices.list):
                        l = (g_hit - vert.Get()).length
                        if(l < 0.01):
                            if(i not in self.PointIdx):
                                if(min == None):
                                    min = l
                                    self.MergeIdx = i
                                else:
                                    if(l < min):
                                        min = l
                                        self.MergeIdx = i
                                g_bMerge = True
                # Edit: Xtra Merge with CTRL + SHIFT
                if ((g_SHIFT == 'PRESS')and (g_CTRL == 'PRESS')):
                    # Merge points around g_hit
                    g_rObject.MergePointsInHit(g_hit)
                # Edit: Xtra Merge with CTRL + SHIFT
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def MergeUpdate(self):
        global g_bMerge

        if(g_bMerge):
            self.Merge(self.MergeIdx, self.PointIdx[0])
            self.MergeIdx = -1
            g_bMerge = False

        self.PointIdx.clear()
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def PointSelected(self):
        return len(self.PointIdx)
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def Undo(self, mode):
        global g_Selection
        
        if(mode == 'SET'):
            self.UndoVL = copy.deepcopy(self.Vertices.list)
            self.UndoTotalVertices = self.Vertices.TotalVertices
            self.UndoFL = copy.deepcopy(self.Faces.list)
            self.UndoNE = self.NbEdges
            self.UndoTotalFaces = self.Faces.TotalFaces
            # Save selection
            self.SaveSel = g_Selection.copy()
        if(mode == 'BACK'):
            self.Vertices.list = copy.deepcopy(self.UndoVL)
            self.Vertices.TotalVertices = self.UndoTotalVertices
            self.Faces.list = copy.deepcopy(self.UndoFL)
            self.NbEdges = self.UndoNE
            self.Faces.TotalFaces = self.UndoTotalFaces
            # Restore selection
            g_Selection = self.SaveSel
    #---------------------------------------------------------------------------------------------------
                
                
    #---------------------------------------------------------------------------------------------------
    def Draw(self):
        global g_region
        global g_rv3d

        global g_PosCutIcon
        global g_PosSizeIcon
        global g_PosCutCTIcon
        
        global g_PosSize2d
        global g_PosCut2d
        global g_bCurveCurrent

        global g_wire_edit_color
        global g_vertex_color
        global g_vertex_sel_color
        global g_vertex_size
        global g_face_color
        global g_face_sel_color
    
        # Transformation de tous les points
        for i in range(self.Vertices.TotalVertices):
            vector3d = self.Vertices.list[i].Get()
            vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
            if(vector2d != None):
                self.Vertices.list[i].Update2d(vector2d)

        # Affichage de la géometrie
        look_at, camera_pos = camera(bpy.context.space_data.region_3d)
        bgl.glEnable(bgl.GL_BLEND)
        for iface in range(self.Faces.TotalFaces):
            face = self.Faces.list[iface]
            vision = camera_pos - self.Vertices.Get(face.VertexIdx[0])
            face.ps = vision.dot(face.Normal)
            if(not g_rv3d.is_perspective):
                face.ps = -face.ps
            if(face.ps < 0):
                if(face.select == True):
                    bgl.glColor4f(g_face_sel_color[0], g_face_sel_color[1], g_face_sel_color[2], g_face_sel_color[3])
                else:            
                    bgl.glColor4f(g_face_color[0], g_face_color[1], g_face_color[2], g_face_color[3])
                try:            
                    if(face.NbVertices == 3):
                        bgl.glBegin(bgl.GL_TRIANGLES)
                        for i in range(face.NbVertices):
                            vertex = self.Vertices.list[face.VertexIdx[i]]
                            bgl.glVertex2f(*vertex.Get2d())
                        bgl.glEnd()
                    else:
                        bgl.glBegin(bgl.GL_QUADS)
                        for i in range(face.NbVertices):
                            vertex = self.Vertices.list[face.VertexIdx[i]]
                            bgl.glVertex2f(*vertex.Get2d())
                        bgl.glEnd()
                except:
                    print("Geometry Draw : Error in GL_QUADS")                

        bgl.glLineWidth(1)
        sface = None
        sCTFace = None
        for iface in range(self.Faces.TotalFaces):
            face = self.Faces.list[iface]
            sface = face
            if((g_PosCutCTIcon != None) and (iface == 0)):
                sCTFace = sface
            if(face.ps < 0):
                bgl.glColor4f(g_wire_edit_color.r, g_wire_edit_color.g, g_wire_edit_color.b, 1.0)
                bgl.glBegin(bgl.GL_LINE_LOOP)
                try:
                    for i in range(face.NbVertices):
                        vertex = self.Vertices.list[face.VertexIdx[i]]
                        bgl.glVertex2f(*vertex.Get2d())
                except:
                    print("Geometry Draw : Error in LineLoop")                
                bgl.glEnd()

                bgl.glPointSize(g_vertex_size)
                bgl.glBegin(bgl.GL_POINTS)
                try:
                    for i in range(face.NbVertices):
                        vertex = self.Vertices.list[face.VertexIdx[i]]
                        if(vertex.select == True):
                            bgl.glColor4f(g_vertex_sel_color.r, g_vertex_sel_color.g, g_vertex_sel_color.b, 1.0)
                        elif(vertex.BS_select == True):
                            bgl.glColor4f(g_vertex_sel_color.r, g_vertex_sel_color.g, g_vertex_sel_color.b, 1.0)
                        else:
                            bgl.glColor4f(g_vertex_color.r, g_vertex_color.g, g_vertex_color.b, 1.0)
                        bgl.glVertex2f(*vertex.Get2d())
                except:
                    print("Geometry Draw : Error in Points")
                bgl.glEnd()

        # Affichage des icones
        if(sface != None):
            if(g_PosCutIcon != None):
                if(sface.ps < 0):
                    if(g_PosSizeIcon != None):
                        g_SizeIcon.DrawIcon()
                        g_PosSize2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, g_PosSizeIcon)
                    g_CutIcon.DrawIcon(True)
                    g_PosCut2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, g_PosCutIcon)
#########################################################################################################



#########################################################################################################
def camera(view):
    look_at = view.view_location
    matrix = view.view_matrix
    pos = camera_position(matrix)
    camera_pos = Vector((pos[0], pos[1], pos[2]))
    
    return look_at, camera_pos
#########################################################################################################



#########################################################################################################
def camera_position(matrix):
    t = (matrix[0][3], matrix[1][3], matrix[2][3])
    r = (
      (matrix[0][0], matrix[0][1], matrix[0][2]),
      (matrix[1][0], matrix[1][1], matrix[1][2]),
      (matrix[2][0], matrix[2][1], matrix[2][2])
    )
    rp = (
      (-r[0][0], -r[1][0], -r[2][0]),
      (-r[0][1], -r[1][1], -r[2][1]),
      (-r[0][2], -r[1][2], -r[2][2])
    )
    output = (
      rp[0][0] * t[0] + rp[0][1] * t[1] + rp[0][2] * t[2],
      rp[1][0] * t[0] + rp[1][1] * t[1] + rp[1][2] * t[2],
      rp[2][0] * t[0] + rp[2][1] * t[1] + rp[2][2] * t[2],
    )
    return output
#########################################################################################################



#########################################################################################################
def Picking(context, self, ray_max = 10000.0, _ray_origin = None, _CTRetopo = False, _fp = None):
    global g_mouse_x
    global g_mouse_y
    global g_bCTRetopo
    global g_CTObj
    
    # get the context arguments
    if(_CTRetopo == False):
        scene = context.scene
        region = context.region
        rv3d = context.region_data
        coord = g_mouse_x, g_mouse_y
    else:
        scene = g_SVScene
        region = g_SVregion
        rv3d = g_SVrv3d
        coord = _fp.x, _fp.y
        
        
    # get the ray from the viewport and mouse
    view_vector = bpy_extras.view3d_utils.region_2d_to_vector_3d(region, rv3d, coord)
    ray_origin = bpy_extras.view3d_utils.region_2d_to_origin_3d(region, rv3d, coord)
    if not rv3d.is_perspective:
        if(abs(view_vector.y) < 1):
            view_vector = -view_vector
    if(_ray_origin != None):
        ray_origin = _ray_origin + view_vector * 0.05
    ray_target = ray_origin + (view_vector * ray_max)
        

    #---------------------------------------------------------------------------------------------------
    def obj_ray_cast(obj, matrix):
        # toggle mode, to force correct drawing
        matrix_inv = matrix.inverted()
        ray_origin_obj = matrix_inv * ray_origin
        ray_target_obj = matrix_inv * ray_target
        success, hit, normal, face_index = obj.ray_cast(ray_origin_obj, ray_target_obj)
        
        if success:
            return hit, normal, face_index
        else:
            return None, None, None

    # cast rays and find the closest object
    best_length_squared = ray_max * ray_max
    best_obj = None
    
    if(g_CTObj == None):    
        for obj in bpy.context.selectable_objects:
            if(obj.type == 'MESH'):
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
    else:                   
        matrix = g_CTObj.matrix_world
        hit, normal, face_index = obj_ray_cast(g_CTObj, matrix)
        if hit is not None:
            hit_world = matrix * hit
            length_squared = (hit_world - ray_origin).length_squared
            if length_squared < best_length_squared:
                best_length_squared = length_squared
                best_obj = g_CTObj
                hits = hit_world
                ns = normal
                fs = face_index
                    
    if(g_bCTRetopo):
        if(best_obj != None):
            g_CTObj = best_obj                    
                
    if best_obj is not None:
        return hits, ns, fs
    else:
        return None, None, None
#########################################################################################################



#########################################################################################################
def draw_callback_px(self, context):
    global g_mouse_x
    global g_mouse_y
    global g_Cursor
    global g_Curve
    global g_rObject
    
    global g_PosCutIcon
    global g_PosSizeIcon
    global g_PosCutCTIcon

    global g_bBSurface
    global g_BSCurve
    
    global g_region
    global g_rv3d
    
    font_id = 0
    #---------------------------------------------------------------------------------------------------
    g_region = bpy.context.region
    g_rv3d = bpy.context.space_data.region_3d
    view_width = context.region.width
    view_height = context.region.height

    blf.size(font_id, 20, 52)
    blf.position(font_id, self.TextPosition + 20, view_height - 50, 0)
    bgl.glColor4f(1.0, 1.0, 1.0, 1.0)
    if(g_bAutoMerge):
        blf.draw(font_id, "AutoMerge ON (A)")
    else:        
        blf.draw(font_id, "AutoMerge OFF (A)")


    bgl.glEnable(bgl.GL_POINT_SMOOTH)
    blf.size(font_id, 20, 72)
    blf.position(font_id, self.TextPosition + 20, view_height - 75, 0)
    if(g_PosCutIcon):
        strInfos = "Cut : " + str(int(g_Cursor.BSize))
        blf.draw(font_id, strInfos)
    if(g_PosCutCTIcon):
        strInfos = "Cut : " + str(int(g_rObject.CTNCut))
        blf.draw(font_id, strInfos)

    # Draw object    
    g_rObject.Draw()
    # Draw curve
    g_Curve.Draw()
    #---------------------------------------------------------------------------------------------------
    if(g_bBSurface):
        for i in range(len(g_BSCurve)):
            g_BSCurve[i].Draw()
    #---------------------------------------------------------------------------------------------------
    if(g_bCTRetopo):
        for i in range(g_CTNLines + 1):
            bgl.glLineWidth(2)
            bgl.glEnable(bgl.GL_LINE_STIPPLE)
            bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
            bgl.glBegin(bgl.GL_LINE_STRIP)
            if(g_CTLines[i][0] != None):
                vector3d = g_CTLines[i][0]
                # Conversion en espace 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
                    
            if(g_CTLines[i][1] != None):
                vector3d = g_CTLines[i][1]
                # Conversion en esapce 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
            bgl.glEnd()
            
            bgl.glPointSize(6)
            bgl.glColor4f(1.0, 0.8, 0.0, 1.0)
            bgl.glBegin(bgl.GL_POINTS)
            if(g_CTLines[i][0] != None):
                vector3d = g_CTLines[i][0]
                # Conversion en espace 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
                    
            if(g_CTLines[i][1] != None):
                vector3d = g_CTLines[i][1]
                # Conversion en esapce 2d
                vector2d = bpy_extras.view3d_utils.location_3d_to_region_2d(g_region, g_rv3d, vector3d)
                # Affichage
                if(vector2d != None):
                    bgl.glVertex2f(*vector2d)
            bgl.glEnd()
            bgl.glDisable(bgl.GL_LINE_STIPPLE)
    #---------------------------------------------------------------------------------------------------
    # Draw Cursor
    g_Cursor.Draw()
    #---------------------------------------------------------------------------------------------------
    # restore opengl defaults
    bgl.glLineWidth(1)
    bgl.glDisable(bgl.GL_POINT_SMOOTH)
    bgl.glPointSize(1)
    bgl.glDisable(bgl.GL_BLEND)
    bgl.glColor4f(0.0, 0.0, 0.0, 1.0)
#########################################################################################################



#########################################################################################################
def getSelection():
    global g_Selection
    
    selobj = bpy.context.active_object
    mesh = selobj.data
    bm = bmesh.from_edit_mesh(mesh)
    bmprev = bm.copy()
    	
    SavIndexPoint = -1

    bmprev.verts.ensure_lookup_table()
    for v in bmprev.verts:
        if v.select:
            NbSelect = 0
            for i in range(len(v.link_edges)):
                bmprev.edges.ensure_lookup_table()
                if(bmprev.edges[v.link_edges[i].index].verts[0].index != v.index):
                    if(bmprev.verts[bmprev.edges[v.link_edges[i].index].verts[0].index].select == True):
                        NbSelect += 1
                        SavIndex = i
                if(bmprev.edges[v.link_edges[i].index].verts[1].index != v.index):
                    if(bmprev.verts[bmprev.edges[v.link_edges[i].index].verts[1].index].select == True):
                        NbSelect += 1
                        SavIndex = i
            if(NbSelect == 1):
                SavIndexPoint = v.index
                break

    if(SavIndexPoint >= 0):
        eList = []
        for e in bmprev.edges:
            if(e.select):
                eList.append([e, False])

        vCurr = bmprev.verts[bmprev.edges[v.link_edges[SavIndex].index].verts[1].index].index
        if(bmprev.verts[bmprev.edges[v.link_edges[SavIndex].index].verts[0].index].index != SavIndexPoint):
            vCurr = bmprev.verts[bmprev.edges[v.link_edges[SavIndex].index].verts[0].index].index
            
        edge = bmprev.verts[bmprev.edges[v.link_edges[SavIndex].index].verts[0].index].index, bmprev.verts[bmprev.edges[v.link_edges[SavIndex].index].verts[1].index].index
        for i, e in enumerate(eList):
            currEdge = e[0].verts[0].index, e[0].verts[1].index
            if(currEdge == edge):
                e[1] = True
                break
        selPoints = []    
        n = len(eList) - 1
        selPoints.append(SavIndexPoint)
        g_Selection.append(SavIndexPoint)

        while(n > 0):
            for i, e in enumerate(eList):
                currEdge = e[0].verts[0].index, e[0].verts[1].index
                if(e[1] == False):
                    if(e[0].verts[0].index == vCurr):
                        if(vCurr not in selPoints):                        
                            selPoints.append(vCurr)
                            g_Selection.append(vCurr)
                        edge = currEdge
                        vCurr = e[0].verts[1].index
                        e[1] = True
                        n -= 1
                    elif(e[0].verts[1].index == vCurr):
                        if(vCurr not in selPoints):                        
                            selPoints.append(vCurr)
                            g_Selection.append(vCurr)
                        edge = currEdge
                        vCurr = e[0].verts[0].index
                        e[1] = True
                        n -= 1
        if(vCurr not in selPoints):                        
            selPoints.append(vCurr)
            g_Selection.append(vCurr)
#########################################################################################################
    
    

#########################################################################################################
class RetopoMT(bpy.types.Operator):
    bl_idname = "mesh.retopomt"
    bl_label = "Retopo MT"
    bl_description = "Multiple tools for retopology."
    bl_options = {'REGISTER', 'UNDO'}
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    @classmethod
    def poll(cls, context):
        ob = context.active_object

        return(ob and ob.type == 'MESH' and context.mode == 'EDIT_MESH')
    #---------------------------------------------------------------------------------------------------
    
    
    #---------------------------------------------------------------------------------------------------
    def modal(self, context, event):
        context.area.tag_redraw()
        
        global g_mouse_x
        global g_mouse_y
        
        global g_normal
        global g_hit
        global g_faceIndex
        global g_bFaceCreating
        
        global g_bBSurface
        global g_BSCurve
        global g_BSCurrent
        global g_BSVs

        global g_CTCurve
        global g_bCTRetopo
        global g_CTNCurve
        global g_CTLines
        global g_CTNLines
        global g_CTObj

        global g_Selection

        global g_bDrawing
        global g_bPtMoving
        global g_bMerge 
        global g_bAutoMerge
        
        global g_PosSizeIcon
        global g_PosCutIcon
        global g_PosCutCTIcon

        global g_SVScene
        global g_SVregion 
        global g_SVrv3d
        
        global g_bLeftBM
        global g_SPACE
        global g_CTRL
        global g_SHIFT
        
        global g_FirstPoint
        global g_EndPoint
        global g_Inverse
        
        g_mouse_x = event.mouse_region_x
        g_mouse_y = event.mouse_region_y
        
        ctrl = False
        
        
        #---------------------------------------------------------------------------------------------------
        if event.type == 'Z':
            if((event.value == "RELEASE") and (event.ctrl)):
                g_PosSizeIcon = g_PosCutIcon = None
                g_rObject.Undo('BACK')
        #---------------------------------------------------------------------------------------------------
        if event.ctrl:
            g_CTRL = 'PRESS'
        else:            
            g_CTRL = 'RELEASE'
        #---------------------------------------------------------------------------------------------------
        if event.type in {'MIDDLEMOUSE', 'WHEELUPMOUSE', 'WHEELDOWNMOUSE'}:
            return {'PASS_THROUGH'}
        #---------------------------------------------------------------------------------------------------
	### Edit: Maya/Silo ALT Navigation
        #---------------------------------------------------------------------------------------------------
        if ((event.type == 'RIGHTMOUSE') and (event.alt)):
            return {'PASS_THROUGH'}
        #---------------------------------------------------------------------------------------------------
        #---------------------------------------------------------------------------------------------------
        if ((event.type == 'LEFTMOUSE') and (event.alt)):
            return {'PASS_THROUGH'}
        #---------------------------------------------------------------------------------------------------
	### Edit: Maya/Silo ALT Navigation
        #---------------------------------------------------------------------------------------------------
        elif (event.type == 'MOUSEMOVE'):
            if((g_Cursor.bBrushSize == False) and (g_Cursor.bCutSize == False)):
                g_hit, g_normal, g_faceIndex = Picking(context, self)
                
            if(self.Icon):
                if(self.SizeIcon):
                    if(g_hit != None):
                        dist = (g_PosSizeIcon - g_hit).length_squared
                        if(dist < 0.01):
                            g_rObject.Undo('BACK')
                            self.Icon = True
                            length = (g_PosCutIcon - g_hit).length
                            if(self.oldLength != 0.0):
                                g_Cursor.BRadius += ((length - self.oldLength) * 20.0)
                            # Compute geometry
                            g_rObject.CreateGeometry()
                            self.oldLength = length
            else:                
                if(g_hit != None):
                    # Show Cursor
                    g_Cursor.Show()
                    # Update cursor
                    g_Cursor.Update()
                    # Update object
                    g_rObject.Update()
                    # Drawing geometry
                    if(g_bDrawing):
                        if(g_bLeftBM == True):
                            if(g_bBSurface):
                                g_BSCurve[g_BSCurrent].Update()
                            elif(g_bCTRetopo):
                                mouse_loc_2d = Vector((g_mouse_x, g_mouse_y))
                                if(self.FirstPoint == None):
                                    self.FirstPoint = g_hit
                                    self.FP2d = mouse_loc_2d
                                    g_CTLines[g_CTNLines][0] = self.FirstPoint
                                    g_CTLines[g_CTNLines][2] = self.FP2d
                                    g_FirstPoint = False
                                self.EndPoint = g_hit
                                self.EP2d = mouse_loc_2d
                                g_CTLines[g_CTNLines][1] = self.EndPoint
                                g_CTLines[g_CTNLines][3] = self.EP2d
                                g_EndPoint = False
                            else:                                
                                g_Curve.Update()
                                g_rObject.PinUpdate()
                else:
                    g_Cursor.Hide()
                    # Si [space] est appuyé, on créé une ligne selon la position du curseur
                    if((g_SPACE == 'PRESS') and (g_bLeftBM == True) and (g_CTNLines >= 0)):
                        mouse_loc_2d = Vector((g_mouse_x, g_mouse_y))
                        mouse_loc_3d = bpy_extras.view3d_utils.region_2d_to_location_3d(g_region, g_rv3d, mouse_loc_2d, self.depth_location)

                        g_hit = mouse_loc_3d
                        if(self.FirstPoint == None):
                            self.FirstPoint = g_hit
                            self.FP2d = mouse_loc_2d
                            g_CTLines[g_CTNLines][0] = self.FirstPoint
                            g_CTLines[g_CTNLines][2] = self.FP2d
                            g_FirstPoint = True
                        self.EndPoint = g_hit
                        self.EP2d = mouse_loc_2d
                        g_EndPoint = True
                        
                        g_CTLines[g_CTNLines][1] = self.EndPoint
                        g_CTLines[g_CTNLines][3] = self.EP2d
                        
                        self.depth_location = mouse_loc_3d
                        
        #---------------------------------------------------------------------------------------------------
        elif(event.type == 'LEFTMOUSE'):
            g_Cursor.bBrushSize = False
            g_Cursor.bCutSize = False
            
            if(event.value == 'PRESS'):
                if(g_hit != None):
                    if((event.ctrl == False) and (event.shift == False)):
                        # Brush Size icon
                        if(g_PosSizeIcon != None):
                            dist = (g_PosSizeIcon - g_hit).length_squared
                            if(dist < 0.01):
                                distCutIcon = (g_PosCutIcon - g_hit).length_squared
                                if(dist < distCutIcon):
                                    self.SizeIcon = True
                                    self.Icon = True

                        # Cut Icon 
                        if(g_PosCutIcon != None):
                            dist = (g_PosCutIcon - g_hit).length_squared
                            if(dist < 0.01):
                                self.CutIcon = True
                                self.Icon = True

                if(self.Icon == False):  
                    # Set Undo object
                    g_rObject.Undo('SET')
                    # Reset object variables
                    g_rObject.ResetCollapse()
                    # Reset curve to draw
                    g_Curve.Reset()
                    
                    g_PosSizeIcon = g_PosCutIcon = g_PosCutCTIcon = None

                    if(event.shift):
                        # Merge points around g_hit
                        g_rObject.MergePointsInHit(g_hit)
                        
                    elif((g_rObject.PointSelected() > 0) and (g_SPACE == 'RELEASE')):
                        # Move point if selected
                        if(event.ctrl):
                            g_PosSizeIcon = g_PosCutIcon = g_PosCutCTIcon = None
                            # Set moving point
                            g_bPtMoving = True
                            # Deselect points 
                            if(len(g_Selection) > 0):
                                for i, vert in enumerate(g_Selection):
                                    g_rObject.Vertices.list[vert].BS_Select(False)
                            g_Selection.clear()
                    else:
                        # Edit: Disable Draw on Tweak (CTRL)
                        if((event.value == 'PRESS') and (event.ctrl == False)):
                        # Edit: Disable Draw on Tweak (CTRL)
                            g_bDrawing = True
                            if((g_rObject.SelectedFace >= 0) and (g_SPACE == 'RELEASE')):
                                g_PosSizeIcon = g_PosCutIcon = g_PosCutCTIcon = None

                                g_rObject.FaceStart = g_rObject.SelectedFace
                                g_bFaceCreating = True
                                # Deselect points 
                                if(len(g_Selection) > 0):
                                    for i, vert in enumerate(g_Selection):
                                        g_rObject.Vertices.list[vert].BS_Select(False)
                                g_Selection.clear()
                            else:
                                if(g_SPACE == 'PRESS'):
                                    if(len(g_Selection) > 0):
                                        # Set mode
                                        g_bBSurface = True
                                        # Create new curve
                                        g_BSCurve.append(cCurve())
                                        la, cpos = camera(bpy.context.space_data.region_3d)
                                        g_BSVs.append(cpos)
                                        g_BSCurrent += 1
                                    else:
                                        g_bCTRetopo = True
                                        g_CTNLines += 1
                                        g_CTLines.append([self.FirstPoint, self.EndPoint, self.FP2d, self. EP2d])
                                        # Save scene variables
                                        g_SVScene = context.scene
                                        g_SVregion = context.region
                                        g_SVrv3d = context.region_data
                                else:
                                    # Deselect points 
                                    if(len(g_Selection) > 0):
                                        for i, vert in enumerate(g_Selection):
                                            g_rObject.Vertices.list[vert].BS_Select(False)
                                    g_Selection.clear()
                                        
                g_bLeftBM = True
            else:
                if(self.SizeIcon == False):
                    if(g_PosCutIcon != None):
                        if(g_hit != None):
                            dist = (g_PosCutIcon - g_hit).length_squared
                            if(dist < 0.02):
                                g_rObject.Undo('BACK')
                                self.CutIcon = True
                                self.Icon = True
                                g_Cursor.BSize += 1
                                # Update geometry
                                g_rObject.CreateGeometry()

                # Create Geometry
                if(g_bDrawing):
                    if((g_bBSurface == False) and (g_bCTRetopo == False)):
                        g_rObject.PinEndUpdate()
                        g_Curve.Homogenize()
                        g_rObject.CreateGeometry()
                        
                if(g_bCTRetopo):
                    if((g_CTLines[g_CTNLines][0] != None) and ((g_CTLines[g_CTNLines][3].x - g_CTLines[g_CTNLines][2].x) != 0.0)):
                        g_CTCurve.append(cCurve())
                        g_CTNCurve += 1
                        
                        div = 0.05
                        incx = 1.0
                        cd = ((g_CTLines[g_CTNLines][3].y - g_CTLines[g_CTNLines][2].y)/(g_CTLines[g_CTNLines][3].x - g_CTLines[g_CTNLines][2].x)) * div
                        fp = g_CTLines[g_CTNLines][2]
                        iter = abs(fp.x - g_CTLines[g_CTNLines][3].x)/div
                        
                        g_Inverse = False
                        if(abs(g_CTLines[g_CTNLines][3].y - g_CTLines[g_CTNLines][2].y) > abs(g_CTLines[g_CTNLines][3].x - g_CTLines[g_CTNLines][2].x)):
                            if(g_CTLines[g_CTNLines][3].y < g_CTLines[g_CTNLines][2].y):
                                g_Inverse = True
                        else:
                            if(g_CTLines[g_CTNLines][3].x < g_CTLines[g_CTNLines][2].x):
                                g_Inverse = True
                        
                        if((fp.x - g_CTLines[g_CTNLines][3].x) > 0.0):
                            cd = -cd
                            incx = -incx
                        nbg_hit = 0
                        while(iter > 0.0):
                            fp.x += incx * div
                            fp.y += cd
                            
                            g_hit, g_normal, g_faceIndex = Picking(context, self, _CTRetopo = True, _fp = fp)
                            if(g_hit != None):
                                g_CTCurve[g_CTNCurve].Update(_context = None, _callsl = self, _CTRetopo = True, _mc = fp)
                                nbg_hit += 1
                            iter -= 1.0                                
                        g_CTCurve[g_CTNCurve].CTUpdate()
                    self.FirstPoint = self.EndPoint = None
                        
                if(g_bMerge):
                    g_rObject.MergeUpdate()                        

                self.Icon = False
                self.CutIcon = False
                self.SizeIcon = False

                g_bFaceCreating = False                    
                g_bDrawing = False                
                g_bLeftBM = False
                g_bPtMoving = False
        #---------------------------------------------------------------------------------------------------
#        elif(event.type == 'RIGHTMOUSE'):
#            if(event.value == 'RELEASE'):
#                if(g_PosCutIcon != None):
#                    if(g_hit != None):
#                        dist = (g_PosCutIcon - g_hit).length_squared
#                        if(dist < 0.02):
#                            self.CutIcon = True
#                            self.Icon = True
#                            if(g_Cursor.BSize > 1.0):
#                                g_Cursor.BSize -= 1.0
#                                if(g_Cursor.BSize < 1.0):
#                                    g_Cursor.BSize = 1.0
#                                    
#                                g_rObject.Undo('BACK')
#                                # Update geometry
#                                g_rObject.CreateGeometry()

#                g_bRightBM = False
#                self.CutIcon = False
#                self.Icon = False
        #---------------------------------------------------------------------------------------------------
        elif(event.type == 'SPACE'):
            if(event.value == 'PRESS'):
                if(g_SPACE == 'RELEASE'):
                    g_SPACE = 'PRESS'
                    g_CTNCurve = -1
                    g_CTCurve.clear()
                    g_FirstPoint = g_EndPoint = True
            else:
                g_SPACE = 'RELEASE'
                if(g_bBSurface):
                    g_rObject.Undo('SET')
                    
                    # Calcul direction
                    fpc = g_BSCurve[0].Curve[0].hit
                    lpc = g_BSCurve[0].Curve[(g_BSCurve[0].NbPoints - 1)].hit
                    # Premier et dernier point de la selection
                    fps = g_rObject.Vertices.Get(g_Selection[0])
                    lps = g_rObject.Vertices.Get(g_Selection[len(g_Selection) - 1])

                    lenf = (fpc - fps).length
                    lenl = (lpc - lps).length
                    lenfl = (fpc - lps).length
                    lenlf = (fps - lpc).length
                    if(((lenf < lenfl) and (lenf < lenlf)) or ((lenl < lenfl) and (lenl < lenlf))):
                        pass
                    else:
                        # Inverse selection
                        g_Selection.reverse()
                    
                    SaveSel = g_Selection.copy()
                    # Create geometry with curves
                    for i in range(len(g_BSCurve)):
                        g_BSCurve[i].Homogenize()
                        g_rObject.CreateBSGeometry(i)
                        
                    g_Selection = SaveSel
                    # Deselect points 
                    if(len(g_Selection) > 0):
                        for i, vert in enumerate(g_Selection):
                            g_rObject.Vertices.list[vert].BS_Select(False)
                    g_Selection.clear()
                
                    g_bBSurface = False
                    g_BSCurrent = -1
                    g_BSCurve.clear()
                    g_BSVs.clear()
                #---------------------------------------------------------------------------------------------------
                if(g_bCTRetopo):
                    for i in range(g_CTNCurve + 1):
                        g_CTCurve[i].Homogenize()
                        g_rObject.CreateCTGeometry(i)

                    g_bCTRetopo = False
                    g_CTObj = None
                    g_CTLines.clear()
                    g_CTNLines = -1
                    g_rObject.CTSel = []
                    self.FirstPoint = self.EndPoint = None
        #---------------------------------------------------------------------------------------------------
        elif event.type == 'A':
            if(event.value == 'RELEASE'):
                # Automerge
                g_bAutoMerge = not g_bAutoMerge
        #---------------------------------------------------------------------------------------------------
        elif event.type == 'W':
            if(event.value == 'RELEASE'):
                if(g_PosCutCTIcon != None):
                    if(g_rObject.CTNCut  > 1.0):
                        g_rObject.CTNCut  -= 1.0
                        if(g_rObject.CTNCut  < 1.0):
                            g_rObject.CTNCut  = 1.0
                            
                        g_rObject.Undo('BACK')
                        # Calcul de la géométrie
                        for i in range(g_CTNCurve + 1):
                            g_rObject.CreateCTGeometry(i)
                if(g_PosCutIcon != None):
                    if(g_Cursor.BSize > 1.0):
                        g_Cursor.BSize -= 1.0
                        if(g_Cursor.BSize < 1.0):
                            g_Cursor.BSize = 1.0
                            
                        g_rObject.Undo('BACK')
                        # Update geometry
                        g_rObject.CreateGeometry()
        #---------------------------------------------------------------------------------------------------
        elif event.type == 'X':
            if(event.value == 'RELEASE'):
                if(g_PosCutCTIcon != None):
                    g_rObject.CTNCut  += 1.0
                        
                    g_rObject.Undo('BACK')
                    # Calcul de la géométrie
                    for i in range(g_CTNCurve + 1):
                        g_rObject.CreateCTGeometry(i)
                if(g_PosCutIcon != None):
                    g_Cursor.BSize += 1.0
                        
                    g_rObject.Undo('BACK')
                    # Update geometry
                    g_rObject.CreateGeometry()
        #---------------------------------------------------------------------------------------------------
        elif event.type == 'V':
            if(event.value == 'RELEASE'):
                g_rObject.Debug()
        #---------------------------------------------------------------------------------------------------
        elif event.type == 'F':
            if(g_SHIFT == 'RELEASE'):
                if(g_Cursor.bCutSize == False):
                    g_Cursor.BrushSize(event)
            else:
                if(g_Cursor.bBrushSize == False):
                    g_Cursor.CutSize(event)                
        #---------------------------------------------------------------------------------------------------
        elif(event.type == 'LEFT_SHIFT'):
            g_SHIFT = event.value
        #---------------------------------------------------------------------------------------------------
        elif event.type in {'Q', 'RIGHTMOUSE'}:
            self.finish()
            bpy.ops.object.mode_set(mode = 'EDIT')
            bpy.ops.mesh.select_mode(use_extend=False, use_expand=False, type='VERT')
            return {'FINISHED'}
                
        #---------------------------------------------------------------------------------------------------
        return {'RUNNING_MODAL'}
    #---------------------------------------------------------------------------------------------------


    #---------------------------------------------------------------------------------------------------
    def invoke(self, context, event):
        if context.object:
            self.bRunning = False
            
            #---------------------------------------------------------------------------------------------------
            global g_Selection
            g_Selection = []
            getSelection()
            
            #---------------------------------------------------------------------------------------------------
            # Get Global variables
            BSize = bpy.context.object.BrushCut
            BRadius = bpy.context.object.BrushSize
            CTNCut = bpy.context.object.CTNCut
            
            AutoMerge = bpy.context.object.AutoMerge
            
            uRO = bpy.context.user_preferences.system.use_region_overlap
            vd = bpy.context.area
            self.TextPosition = 0
            if(uRO):
                self.TextPosition = vd.regions[1].width
            
            #---------------------------------------------------------------------------------------------------
            # Search objects in Edit mode
            obE = None
            for o in bpy.context.selected_objects:
                if(o.mode == 'EDIT'):
                    obE = o
                else:
                    o.select = False        
                
            #---------------------------------------------------------------------------------------------------
            bpy.ops.object.mode_set(mode='OBJECT')
            
            o = obE
            self.obName     = o.show_name
            self.obAxis     = o.show_axis
            self.obWire     = o.show_wire
            self.obEdge     = o.show_all_edges
            self.obBounds   = o.show_bounds
            self.obTSpace   = o.show_texture_space
            self.obTransp   = o.show_transparent
            self.obX_Ray    = o.show_x_ray
            self.obDrawType = o.draw_type
            self.OBName     = o.name
            self.use_smooth = False
            if(len(o.data.polygons) > 0):
                self.use_smooth = o.data.polygons[0].use_smooth
            
            self.Modifiers = []
            prp = []
            self.prpLst = {}
            for mAO in o.modifiers:
                self.Modifiers.append((mAO.name, mAO.type))
                mDst = o.modifiers.get(mAO.name, None)
                properties = [p.identifier for p in mAO.bl_rna.properties
                              if not p.is_readonly]
                prp.clear()
                for prop in properties:
                    prp.append((prop, getattr(mAO, prop)))
                self.prpLst[mAO.name] = prp.copy()
            #---------------------------------------------------------------------------------------------------
            global g_rObject
            g_rObject = cObject()
            g_rObject.CTNCut = CTNCut

            # Conversion
            try:
                selectedVertices = False
                for n, v in enumerate(o.data.vertices):
                    g_rObject.AddVertex(cVertex(v.co.x, v.co.y, v.co.z, BSelect = v.select))
                    if(v.select == True):
                        selectedVertices = True
                    g_rObject.NbEdges += 1
                    
                for n, f in enumerate(o.data.polygons):
                    if(len(f.vertices) == 4):
                        g_rObject.AddFace(cFace((f.vertices[0], f.vertices[3], f.vertices[2], f.vertices[1]))) 
                    if(len(f.vertices) == 3):
                        g_rObject.AddFace(cFace((f.vertices[0], f.vertices[1], f.vertices[2]))) 
                        
                bpy.ops.object.mode_set(mode='OBJECT')
                bpy.data.objects[self.OBName].select = True
                bpy.ops.object.delete(use_global = False)
            except:
                self.report({'WARNING'}, "Error in execution")
                return {'CANCELLED'}
            #---------------------------------------------------------------------------------------------------

            #---------------------------------------------------------------------------------------------------
            # Variables
            self.Icon = False
            self.CutIcon = False
            self.SizeIcon = False
            self.oldLength = 0.0
            self.depth_location = Vector((0.0, 0.0, 0.0))
            self.last_ml = Vector((0.0, 0.0, 0.0))
            
            #---------------------------------------------------------------------------------------------------
            global g_Curve
            g_Curve = cCurve()
            
            #---------------------------------------------------------------------------------------------------
            global g_BSCurve
            g_BSCurve = []
            global g_BSVs
            g_BSVs = []

            #---------------------------------------------------------------------------------------------------
            # Contour object
            global g_CTObj
            g_CTObj = None
            global g_CTCurve
            g_CTCurve = []
            global g_CTNCurve
            g_CTNCurve = -1
            global g_bCTRetopo
            g_bCTRetopo = False
            global g_CTLines
            g_CTLines = []
            global g_CTNLines
            g_CTNLines = -1
            self.FirstPoint = None
            self.EndPoint = None
            self.FP2d = None
            self.EP2d = None

            global g_FirstPoint
            g_FirstPoint = True
            global g_EndPoint
            g_EndPoint = True
            global g_Inverse
            g_Inverse = False

            #---------------------------------------------------------------------------------------------------
            global g_normal
            g_normal = Vector()
            global g_hit
            g_hit = Vector()
            global g_faceIndex
            g_faceIndex = -1
            
            #---------------------------------------------------------------------------------------------------
            global g_bAutoMerge
            g_bAutoMerge = AutoMerge
            global g_bDrawing
            g_bDrawing = False

            #---------------------------------------------------------------------------------------------------
            global g_Cursor
            g_Cursor = cCursor(10.0, 0.05)
            global g_CutIcon
            g_CutIcon = cCursor(45.0, 0.0155)
            global g_SizeIcon
            g_SizeIcon = cCursor(45.0, 0.0175)
            
            g_Cursor.BSize = BSize
            g_Cursor.BRadius = BRadius


            #---------------------------------------------------------------------------------------------------
            global g_PosCutIcon
            g_PosCutIcon = None
            global g_PosSizeIcon
            g_PosSizeIcon = None
            global g_PosCutCTIcon
            g_PosCutCTIcon = None

            #---------------------------------------------------------------------------------------------------
            global g_PosSize2d
            g_PosSize2d = (-1, -1)
            global g_PosCut2d
            g_PosCut2d = (-1, -1)
            
            global g_bMerge
            g_bMerge = False
            
            global g_bPtMoving
            g_bPtMoving = False
            global g_bFaceMoving
            g_bFaceMoving = False
            global g_bFaceCreating
            g_bFaceCreating = False

            global g_bBSurface
            g_bBSurface = False
            global g_bCurveCurrent
            g_bCurveCurrent = False
            global g_BSCurrent
            g_BSCurrent = -1

            global g_bLeftBM
            g_bLeftBM = False
            global g_bRightBM
            g_bRightBM = False
            global g_SPACE
            g_SPACE = "RELEASE"
            global g_CTRL
            g_CTRL = "RELEASE"
            global g_SHIFT
            g_SHIFT = "RELEASE"
            global g_SavLens
            g_SavLens = 85.0

            global g_SavPersp
            g_SavPersp = bpy.context.space_data.region_3d.is_perspective
            if(bpy.context.space_data.region_3d.is_perspective == False):
                global g_SavLens
                g_SavLens = bpy.context.space_data.lens

#                bpy.context.space_data.lens = 250.0

                for area in bpy.context.screen.areas:
                    if area.type == "VIEW_3D":
                        break

                for region in area.regions:
                    if region.type == "WINDOW":
                        break
                space = area.spaces[0]
                contextc = bpy.context.copy()
                contextc['area'] = area
                contextc['region'] = region
                contextc['space_data'] = space
                bpy.ops.view3d.view_persportho(contextc, 'EXEC_DEFAULT')
        
            #---------------------------------------------------------------------------------------------------
            theme = bpy.context.user_preferences.themes['Default']
            
            global g_wire_edit_color
            global g_vertex_color
            global g_vertex_sel_color
            global g_vertex_size
            global g_face_color
            global g_face_sel_color
            
            g_wire_edit_color = theme.view_3d.wire_edit
            g_vertex_color    = theme.view_3d.vertex
            g_vertex_sel_color= theme.view_3d.vertex_select
            g_vertex_size     = theme.view_3d.vertex_size
            g_face_color      = theme.view_3d.face
            g_face_sel_color  = theme.view_3d.face_select
            
            # the arguments we pass the the callback
            args = (self, context)
            # Add the region OpenGL drawing callback
            # draw in view space with 'POST_VIEW' and 'PRE_VIEW'
            self._handle = bpy.types.SpaceView3D.draw_handler_add(draw_callback_px, args, 'WINDOW', 'POST_PIXEL')

            # Hide cursor
            bpy.context.window.cursor_modal_set("NONE")

            # Display the operator's instructions in the active area's header.
            context.area.header_text_set(
                "Draw: LMB     W/X: Nb cuts     Move vertex: Ctrl+LMB     Merge Points : SHIFT+LMB     Extrude selecting: Space+LMB     Brush Size: F     Brush Cut Size: SHIFT+F     AutoMerge: A" +
                "         Undo: Ctrl+Z    Confirm: RMB/Enter"
            )

            context.window_manager.modal_handler_add(self)

            return {'RUNNING_MODAL'}
        else:
            self.report({'WARNING'}, "No active object, could not finish")
            return {'CANCELLED'}
#########################################################################################################



#########################################################################################################
    def finish(self):
        # Create mesh
        mesh = bpy.data.meshes.new(self.OBName)
        ob = bpy.data.objects.new(self.OBName, mesh)
        ob.location = Vector()
     
        scn = bpy.context.scene
        scn.objects.link(ob)
        scn.objects.active = ob
        ob.select = True

        ob.show_name          = self.obName
        ob.show_axis          = self.obAxis
        ob.show_wire          = self.obWire
        ob.show_all_edges     = self.obEdge
        ob.show_bounds        = self.obBounds
        ob.show_texture_space = self.obTSpace
        ob.show_transparent   = self.obTransp
        ob.show_x_ray         = self.obX_Ray
        ob.draw_type          = self.obDrawType
    
        if(g_rObject.Vertices.TotalVertices > 0):
            verts = []
            # Conversion du maillage 
            for i, v in enumerate(g_rObject.Vertices.list):
                vt = (v.x, v.y, v.z)
                verts.append(vt)
            faces = []
            ft = []
            for i, f in enumerate(g_rObject.Faces.list):
                ft.clear()
                if(len(f.VertexIdx) == 4):
                    ft.append(f.VertexIdx[0])
                    ft.append(f.VertexIdx[3])
                    ft.append(f.VertexIdx[2])
                    ft.append(f.VertexIdx[1])
                if(len(f.VertexIdx) == 3):
                    ft.append(f.VertexIdx[0])
                    ft.append(f.VertexIdx[2])
                    ft.append(f.VertexIdx[1])

                faces.append(ft.copy())
            
            mesh.from_pydata(verts, [], faces)
            mesh.update(calc_edges = True)    

            for m in self.Modifiers:
                mDst = ob.modifiers.new(m[0], m[1])
                for i in range(len(self.prpLst[m[0]])):
                    setattr(mDst, self.prpLst[m[0]][i][0], self.prpLst[m[0]][i][1])

            for p in mesh.polygons:
                p.use_smooth = self.use_smooth                    

        #---------------------------------------------------------------------------------------------------
        self.bRunning = False     

        #---------------------------------------------------------------------------------------------------
        bpy.ops.object.mode_set(mode='EDIT')
        bpy.ops.mesh.remove_doubles()
        bpy.ops.mesh.select_all()
     
        #---------------------------------------------------------------------------------------------------
        bpy.context.object.BrushCut     = g_Cursor.BSize
        bpy.context.object.BrushSize    = g_Cursor.BRadius
        bpy.context.object.CTNCut       = g_rObject.CTNCut
        bpy.context.object.AutoMerge    = g_bAutoMerge
        


        #global g_SavLens
#        global g_SavPersp
        bpy.context.space_data.lens = g_SavLens

        if(g_SavPersp == False):
            for area in bpy.context.screen.areas:
                if area.type == "VIEW_3D":
                    break

            for region in area.regions:
                if region.type == "WINDOW":
                    break
            space = area.spaces[0]
            context = bpy.context.copy()
            context['area'] = area
            context['region'] = region
            context['space_data'] = space
            bpy.ops.view3d.view_persportho(context, 'EXEC_DEFAULT')

        
        bpy.types.SpaceView3D.draw_handler_remove(self._handle, 'WINDOW')
        bpy.context.window.cursor_modal_set("DEFAULT")

        # Restore la vue par défaut
        bpy.context.area.header_text_set()
#########################################################################################################




#########################################################################################################
classes = [RetopoMT]
addon_keymaps = []
#########################################################################################################


#########################################################################################################
def register():
    bpy.utils.register_class(RetopoMTPrefs) 
    # add operator
    for c in classes:
        bpy.utils.register_class(c)

    bpy.types.Object.BrushSize  = bpy.props.FloatProperty(default = 1.0, min = 0.0, max = 100.0)
    bpy.types.Object.BrushCut   = bpy.props.FloatProperty(default = 6.0, min = 0.0, max = 100.0)
    bpy.types.Object.CTNCut     = bpy.props.FloatProperty(default = 6.0, min = 0.0, max = 100.0)
    bpy.types.Object.AutoMerge  = bpy.props.BoolProperty(default = True)

    # add keymap entry
    kcfg = bpy.context.window_manager.keyconfigs.addon
    if kcfg:
        km = kcfg.keymaps.new(name='Mesh', space_type='EMPTY')
        kmi = km.keymap_items.new("mesh.retopomt", 'X', 'PRESS', ctrl=True, alt=True, shift=True)
        addon_keymaps.append((km, kmi))
#########################################################################################################


#########################################################################################################
def unregister():
    bpy.utils.unregister_class(RetopoMTPrefs) 
    # remove keymap entry
    for km, kmi in addon_keymaps:
        km.keymap_items.remove(kmi)
    addon_keymaps.clear()
    
    # remove operator and preferences
    for c in reversed(classes):
        bpy.utils.unregister_class(c)
#########################################################################################################


#########################################################################################################
if __name__ == "__main__":
    register()
