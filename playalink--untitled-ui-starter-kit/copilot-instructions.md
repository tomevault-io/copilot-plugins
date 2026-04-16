## untitled-ui-starter-kit

> Detailed Figma-to-React mapping for the Untitled UI Modal/Dialog component.

---
---

# Modal Component

Detailed Figma-to-React mapping for the Untitled UI Modal/Dialog component.

## Import

```tsx
import {
  Modal,
  ModalHeader,
  ModalContent,
  ModalFooter,
  ModalTitle,
  ModalDescription,
  ModalIcon,
  ModalClose
} from '@/components/untitled-ui';
```

Source: `components/untitled-ui/base/modal/`

## Props API

```tsx
interface ModalProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  size?: 'sm' | 'md' | 'lg' | 'xl';
  position?: 'center' | 'top';  // default: "center"
  fullScreen?: boolean;
  children: ReactNode;
}

interface ModalHeaderProps {
  children: ReactNode;
}

interface ModalIconProps {
  icon: ReactNode;
  color?: 'brand' | 'error' | 'warning' | 'success';
}
```

## Figma → React Mapping

### Size
| Figma `Size` | React |
|--------------|-------|
| `sm` | `size="sm"` |
| `md` | `size="md"` |
| `lg` | `size="lg"` |
| `xl` | `size="xl"` |

### Type/Position
| Figma `Type` | React |
|--------------|-------|
| `Centered` | `position="center"` (default) |
| `Top aligned` | `position="top"` |
| `Full screen` | `fullScreen` |

## Figma Structure → React Composition

```yaml
# Figma structure
- Modal
  - Modal header
    - Featured icon
    - Close button (X)
  - Modal content
    - Title text
    - Description text
    - [Custom content]
  - Modal footer
    - Button (Cancel)
    - Button (Confirm)
```

```tsx
// React structure
<Modal open={open} onOpenChange={setOpen}>
  <ModalHeader>
    <ModalIcon icon={<AlertCircle />} color="error" />
    <ModalClose />
  </ModalHeader>
  <ModalContent>
    <ModalTitle>Delete item?</ModalTitle>
    <ModalDescription>
      This action cannot be undone.
    </ModalDescription>
  </ModalContent>
  <ModalFooter>
    <Button color="secondary" onPress={() => setOpen(false)}>
      Cancel
    </Button>
    <Button color="primary-destructive" onPress={handleDelete}>
      Delete
    </Button>
  </ModalFooter>
</Modal>
```

## Usage Examples

### Basic Confirmation Modal
```tsx
const [open, setOpen] = useState(false);

<Button onPress={() => setOpen(true)}>Open Modal</Button>

<Modal open={open} onOpenChange={setOpen}>
  <ModalHeader>
    <ModalClose />
  </ModalHeader>
  <ModalContent>
    <ModalTitle>Confirm action</ModalTitle>
    <ModalDescription>Are you sure you want to proceed?</ModalDescription>
  </ModalContent>
  <ModalFooter>
    <Button color="secondary" onPress={() => setOpen(false)}>Cancel</Button>
    <Button color="primary" onPress={handleConfirm}>Confirm</Button>
  </ModalFooter>
</Modal>
```

### With Icon
```tsx
import { AlertTriangle } from '@untitledui/icons';

<Modal open={open} onOpenChange={setOpen}>
  <ModalHeader>
    <ModalIcon icon={<AlertTriangle />} color="warning" />
    <ModalClose />
  </ModalHeader>
  <ModalContent>
    <ModalTitle>Warning</ModalTitle>
    <ModalDescription>This will affect all users.</ModalDescription>
  </ModalContent>
  <ModalFooter>
    <Button color="secondary" onPress={() => setOpen(false)}>Cancel</Button>
    <Button color="primary" onPress={handleProceed}>Proceed</Button>
  </ModalFooter>
</Modal>
```

### Destructive Action
```tsx
import { Trash2 } from '@untitledui/icons';

<Modal open={open} onOpenChange={setOpen}>
  <ModalHeader>
    <ModalIcon icon={<Trash2 />} color="error" />
    <ModalClose />
  </ModalHeader>
  <ModalContent>
    <ModalTitle>Delete project?</ModalTitle>
    <ModalDescription>
      This will permanently delete the project and all associated data.
    </ModalDescription>
  </ModalContent>
  <ModalFooter>
    <Button color="secondary" onPress={() => setOpen(false)}>Cancel</Button>
    <Button color="primary-destructive" onPress={handleDelete}>Delete</Button>
  </ModalFooter>
</Modal>
```

### Different Sizes
```tsx
<Modal open={open} onOpenChange={setOpen} size="lg">
  {/* Larger modal for more content */}
</Modal>
```

## Figma Data Example

```json
{
  "name": "Modal",
  "componentProperties": {
    "Size": { "value": "md" },
    "Type": { "value": "Centered" }
  },
  "children": [
    { "name": "Modal header", "children": [...] },
    { "name": "Modal content", "children": [...] },
    { "name": "Modal footer", "children": [...] }
  ]
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PlayaLink) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
