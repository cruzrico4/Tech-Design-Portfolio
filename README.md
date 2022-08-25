# Tech Design Portfolio
## Automation Projects
### Streamlined Icon Generation
As a Technical Designer, a large part of my job is to design and implement tools that streamline art workflows. This mini-project allowed artists to automatically generate icons for hundreds of objects at the click of a button, saving countless hours of work for them, had they done each icon manually.

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
### Automated Asset Importing
This project set out to address the problem of getting the sheer volume of assets the team had created into the editor, since it had no built-in batch importing function.

### Automated Avatar Rigging
This problem is similar to the last one, but added on the functionality to directly apply .fbx objects to the body parts of our characters in engine.

Since engine didn't natively allow importing single .fbx character model and rigs, we needed a way to quickly export characters piecemeal and reconstruct them in engine, while retaining joint data.

My solution was to batch export character body parts as .fbx files and write their joint data to .json. I then used an AutoIt script to automate clicks that imported each body part to its correct position, and applied the joint data from the .json file.

Again, this solution saved innumerable hours of rigging by hand, and allowed artists to see their rigged models in-engine extremely quickly, and allowed them to make changes as needed without wasting time.

An example of a character model that needed to have its body parts, as well as position, size, and joint data updated:

![Avater that needed to be imported]([https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/AvatarBuilderSpeedUp.gif](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/NewAvatar.png))

The following gif shows the new model's position, scale, and joint data being updated automatically:

![Automated Avatar Rigging Gif](https://github.com/cruzrico4/Tech-Design-Portfolio/blob/main/Projects/Automation/Media/AvatarBuilderSpeedUp.gif)
