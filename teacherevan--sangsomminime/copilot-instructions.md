## sangsomminime

> globs: *.py, Assets/**/*.py

---
description: 
globs: *.py, Assets/**/*.py
alwaysApply: false
---

# Blender Python Development Standards

## Code Structure Requirements

### Namespace and Class Organization
- **Always use explicit modules** following project structure: `SangsomMiniMe.{Folder}.{Subfolder}`
- **Single class per file** with matching filename
- **Use docstrings** to document classes and methods

### Blender-Specific Patterns
- **Operator lifecycle methods** in standard order: poll, invoke, execute, modal
- **Use property decorators** for Blender properties
- **Use handlers** for loose coupling: `bpy.app.handlers` for event handling
- **Implement proper cleanup** for classes managing resources

### Performance and Memory
- **Cache object references** at initialization, not in modal loops
- **Use batch operations** for frequently modified objects
- **Avoid unnecessary context updates** with proper context overrides
- **Implement proper null checking** with Python's idiomatic patterns

## Error Handling and Logging

### Exception Management
```python
try:
    # Risky operation
    character_data = load_character_data(student_id)
except Exception as ex:
    print(f"Failed to load character data for {student_id}: {ex}")
    character_data = create_default_character_data(student_id)
```

### Debug Logging Standards
- **Use print() for development** information
- **Use logging.warning()** for non-critical issues  
- **Use logging.error()** for critical failures
- **Include context information** in all log messages

## Examples

<example>
Proper Blender Python class structure:
```python
import bpy
from bpy.types import Operator, Panel
from bpy.props import FloatProperty, StringProperty

class CharacterCustomizationOperator(Operator):
    """Operator for character customization"""
    bl_idname = "sangsom.character_customize"
    bl_label = "Customize Character"
    bl_options = {'REGISTER', 'UNDO'}
    
    # Properties
    eye_size: FloatProperty(
        name="Eye Size",
        default=1.0,
        min=0.5,
        max=2.0
    )
    
    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    
    def execute(self, context):
        try:
            self.update_character(context)
            return {'FINISHED'}
        except Exception as ex:
            self.report({'ERROR'}, f"Failed to update character: {ex}")
            return {'CANCELLED'}
    
    def update_character(self, context):
        obj = context.active_object
        if obj:
            obj.scale = (self.eye_size, self.eye_size, self.eye_size)

class CharacterCustomizationPanel(Panel):
    """Panel for character customization UI"""
    bl_label = "Character Customization"
    bl_idname = "SANGSOM_PT_character_customization"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "Sangsom Mini-Me"
    
    def draw(self, context):
        layout = self.layout
        layout.operator("sangsom.character_customize")

def register():
    bpy.utils.register_class(CharacterCustomizationOperator)
    bpy.utils.register_class(CharacterCustomizationPanel)

def unregister():
    bpy.utils.unregister_class(CharacterCustomizationOperator)
    bpy.utils.unregister_class(CharacterCustomizationPanel)

if __name__ == "__main__":
    register()
```
</example>

<example type="invalid">
Avoid poor Blender Python practices:
```python
# ❌ DON'T: Bad Blender practices
class BadCharacterScript:
    eye_size = 1.0  # Global variable instead of property
    
    def update(self):
        # Finding objects every frame - performance killer
        character = bpy.data.objects.get("Character")
        
        # No error handling
        character.scale = (self.eye_size, self.eye_size, self.eye_size)
```
</example>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TeacherEvan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
