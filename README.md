# Tech Design Portfolio
<details>
    
<summary>
    
## Automation Projects (Python, AutoIt)
</summary>
    
### Streamlined Icon Generation
    
<details>
    
<summary>
As a Technical Designer, a large part of my job is to design and implement tools that streamline art workflows...
</summary>

This mini-project allowed artists to automatically generate icons for hundreds of objects at the click of a button, saving countless hours of work for them, had they done each icon manually.


I would later automate the importing process as well, since batch importing was not supported in our engine.

![Automated icon rendering in Blender](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/ScrollingAutomatedIcons.gif)

```Python
########################################################
# Last updated: 08/22/2022
# 
# Renders transparent png images of each mesh objects 
# in the current scene using the current active camera
# and exports to a given directory
# 
########################################################
import bpy
import os

#Get file path for prefixing exported .obj's
file_path = bpy.data.filepath
filepath_split = os.path.split(file_path) #list with [path, name]
desktop = os.path.join(os.path.join(os.environ['USERPROFILE']), 'Desktop')
#Export path
abs_path = desktop + "\\ReworldProjects\\MobileCreationTool\\Guns\\"

all_obj = bpy.context.scene.objects

#Uncheck all objects render
for obj in all_obj:
    if(obj.type == "MESH"):
        obj.hide_render = True

#Set BG transparent
bpy.context.scene.render.film_transparent = True

for obj in all_obj:
    if(obj.type == "MESH"):
        prefix_string = str.split(str(obj.users_collection[0].name), ".")[0]
        #construct export path
        export_path = abs_path + bpy.context.scene.name + "\\" + "BeautyRenders" + "\\" + prefix_string + "\\"
        ext = ".png"
        #make path if it doesn't exist
        if not os.path.isdir(export_path):
            os.makedirs(export_path)
        bpy.ops.object.select_all(action="DESELECT")
        obj.select_set(True)
        obj.hide_render = False
        bpy.context.scene.render.filepath = export_path + str.split(obj.name,".")[0] + ext
        bpy.ops.render.render(write_still = 1)
        obj.hide_render = True
```
</details>

----
    
### Exporting Assets With Smallest Collective Bounds
    
<details>
    
<summary>
Many assets needed to be exported individually, but also needed to have the same origin, which this project solved...
    
</summary>
    
This was an interesting problem that allowed me to explore more ways to automate processing assets to optimize their usability for specific purposes in engine. The following image shows the bounds needed, based on many individual objects (the roof, chimney, walls, door, and steps), in order to place them all at one location and maintain their visual offset from one another.
    
![Smallest bounds of multiple objects](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/AssetBounds.jpeg)

    
<details>
    
<summary>
The Code:

</summary>

```Python
import bpy
import os
from pathlib import Path

maxXName = "NONE"
maxYName = "NONE"
maxZName = "NONE"
minXName = "NONE"
minYName = "NONE"
minZName = "NONE"
maxX = 0
maxY = 0
maxZ = 0
minX = 999999999999
minY = 999999999999
minZ = 999999999999
tinyScale = 0.00001
boundObs = []
expObs = []
desktop = os.path.join(os.path.join(os.environ['USERPROFILE']), 'Desktop')
prefix_string = bpy.context.active_object.users_collection[0].name

#maxZ,maxY,maxX,minZ,minY,minX
for obj in bpy.context.selected_objects:
    expObs.append(obj)
    if obj is not None and obj.type == "MESH":
        mesh = obj.data
        for vert in mesh.vertices:
            globalVert = obj.matrix_world @ vert.co
            if maxZ < globalVert[2]:
                maxZ = globalVert[2]
                maxZName = obj.name
            if maxY < globalVert[1]:
                maxY = globalVert[1]
                maxYName = obj.name
            if maxX < globalVert[0]:
                maxX = globalVert[0]
                maxXName = obj.name
            if minZ > globalVert[2]:
                minZ = globalVert[2]
                minZName = obj.name
            if minY > globalVert[1]:
                minY = globalVert[1]
                minYName = obj.name
            if minX > globalVert[0]:
                minX = globalVert[0]
                minXName = obj.name

print(maxXName," has a bound at X = ",maxX)
print(maxYName," has a bound at Y = ",maxY)
print(maxZName," has a bound at Z = ",maxZ)
bpy.ops.mesh.primitive_cube_add(location=(maxX,maxY,maxZ))
bpy.context.active_object.name = "UpperBound"
bpy.context.active_object.scale.x = tinyScale
bpy.context.active_object.scale.y = tinyScale
bpy.context.active_object.scale.z = tinyScale
boundObs.append(bpy.data.objects["UpperBound"])

print(minXName," has a bound at X = ",minX)
print(minYName," has a bound at Y = ",minY)
print(minZName," has a bound at Z = ",minZ)
bpy.ops.mesh.primitive_cube_add(location=(minX,minY,minZ))
bpy.context.active_object.name = "LowerBound"
bpy.context.active_object.scale.x = tinyScale
bpy.context.active_object.scale.y = tinyScale
bpy.context.active_object.scale.z = tinyScale
boundObs.append(bpy.data.objects["LowerBound"])

ctx = bpy.context.copy()

# one of the objects to join
ctx['active_object'] = boundObs[0]

ctx['selected_objects'] = ctx["selected_editable_objects"] = boundObs

bpy.ops.object.join(ctx)

##========================================================================
##========================================================================
##========================================================================

#Get file path for prefixing exported .obj's
file_path = bpy.data.filepath
filepath_split = os.path.split(file_path) #list with [path, name]
print(prefix_string)

#construct export path
export_path = desktop + "\\ReworldObjects\\MobileCreationTool\\Objects\\Props" + "\\" + prefix_string + "\\"
ext = ".obj"

#make path if not exist
if not os.path.isdir(export_path):
    os.makedirs(export_path)

#export all selected objects to export_path
for obj in expObs: 
    bpy.ops.object.select_all(action="DESELECT")
    obj.select_set(True)
    bpy.data.objects["UpperBound"].select_set(True)
    full_path = export_path + prefix_string + "_" + str.split(obj.name,".")[0] + ext
    print(full_path)
    bpy.ops.export_scene.obj(filepath=full_path,use_selection=True)
bpy.ops.object.select_all(action="DESELECT")
bpy.data.objects["UpperBound"].select_set(True)
bpy.ops.object.delete()
```
</details>
    
</details>
        
----
    
### Automated Asset Importing

<details>
    
<summary>
This project set out to address the problem of getting the sheer volume of assets the team had created into the editor, since it had no built-in batch importing function...
</summary>

</details>
    
----

### Automated Avatar Rigging

<details>
<summary>
This problem is similar to the last one, but added on the functionality to directly apply .fbx objects to the body parts of our characters in engine, and apply external data to the character rig in order to achieve a perfect 1:1 replacement...
</summary>

Since engine didn't natively allow importing single .fbx character model and rigs, we needed a way to quickly export characters piecemeal and reconstruct them in engine, while retaining joint data.

My solution was to batch export character body parts as .fbx files and write their joint data to .json. I then used an AutoIt script to automate clicks that imported each body part to its correct position, and applied the joint data from the .json file.

Again, this solution saved innumerable hours of rigging by hand, and allowed artists to see their rigged models in-engine extremely quickly, and allowed them to make changes as needed without wasting time.

An example of a character model that needed to have its body parts, as well as position, size, and joint data updated:

<img src="https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/NewAvatar.png" width="512px" height="512px" />
<!---![Avatar that needed to be imported](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/NewAvatar.png =512x512)--->

The following gif shows the new model's .fbx body parts, position, scale, and joint data being updated automatically using AutoIt in lieu of built-in functionality:

![Automated Avatar Rigging Gif](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/AvatarBuilder.gif)
</details>

----

</details>

<details>

<summary>
    
## Game Features (Lua)

</summary>

</details
