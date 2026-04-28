## regimento-interno

> TITLE: Implementing Individual Test Cases with Descriptive Names

TITLE: Implementing Individual Test Cases with Descriptive Names
DESCRIPTION: Demonstrates how to create individual test cases with descriptive names that clearly state the expected behavior being tested.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/core/testing/best-practices.md#2025-04-16_snippet_3

LANGUAGE: jsx
CODE:
```
 // src/components/button/test/basic/button.e2e.ts

import { configs, test } from '@utils/test/playwright';

configs().forEach(({ title }) => {
  test.describe(title('button: disabled state'), () => {
    test('should not have any visual regressions', async ({ page }) => {
      ...
    });
  });
});
```

----------------------------------------

TITLE: Implementing Accessible Checkbox with VoiceOver Support in TSX
DESCRIPTION: TSX implementation for a checkbox that works properly with VoiceOver. Sets required ARIA attributes on both host and input elements for proper accessibility.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_18

LANGUAGE: tsx
CODE:
```
render() {
  const { checked, disabled } = this;

  return (
    <Host
      aria-checked={`${checked}`}
      aria-hidden={disabled ? 'true' : null}
      role="checkbox"
    >
      <input
        type="checkbox"
      />
      ...
    </Host>
  );
}
```

----------------------------------------

TITLE: Defining the Ionic Searchbar API
DESCRIPTION: Comprehensive documentation of the ion-searchbar component API including properties, methods, events, and CSS custom properties. The searchbar provides functionality for user input with features like autocomplete, debounce, and clear/cancel buttons.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_33

LANGUAGE: typescript
CODE:
```
ion-searchbar,prop,autocomplete,"name" | "email" | "tel" | "url" | "on" | "off" | "honorific-prefix" | "given-name" | "additional-name" | "family-name" | "honorific-suffix" | "nickname" | "username" | "new-password" | "current-password" | "one-time-code" | "organization-title" | "organization" | "street-address" | "address-line1" | "address-line2" | "address-line3" | "address-level4" | "address-level3" | "address-level2" | "address-level1" | "country" | "country-name" | "postal-code" | "cc-name" | "cc-given-name" | "cc-additional-name" | "cc-family-name" | "cc-number" | "cc-exp" | "cc-exp-month" | "cc-exp-year" | "cc-csc" | "cc-type" | "transaction-currency" | "transaction-amount" | "language" | "bday" | "bday-day" | "bday-month" | "bday-year" | "sex" | "tel-country-code" | "tel-national" | "tel-area-code" | "tel-local" | "tel-extension" | "impp" | "photo",'off',false,false
```

----------------------------------------

TITLE: CSS Differences: Scoped vs Shadow DOM in Ionic Components
DESCRIPTION: This snippet illustrates the CSS differences when converting from scoped to shadow DOM. It covers targeting host elements, slotted children, and host-context scenarios.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_26

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host(.ion-color)::slotted(ion-segment-button)

/* IN SHADOW*/
:host(.ion-color) ::slotted(ion-segment-button)
```

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host-context(ion-toolbar.ion-color):not(.ion-color) {

/* IN SHADOW */
:host-context(ion-toolbar.ion-color):host(:not(.ion-color))  {
```

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host-context(ion-toolbar:not(.ion-color)):not(.ion-color)::slotted(ion-segment-button) {

/* IN SHADOW*/
:host-context(ion-toolbar:not(.ion-color)):host(:not(.ion-color)) ::slotted(ion-segment-button) {
```

----------------------------------------

TITLE: Implementing Accessible Labels for Form Controls in TSX
DESCRIPTION: TSX implementation showing how to properly handle labels for form controls, inheriting ARIA attributes from host elements for better accessibility.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_20

LANGUAGE: tsx
CODE:
```
import { Prop } from '@stencil/core';
import { inheritAttributes } from '@utils/helpers';
import type { Attributes } from '@utils/helpers';

...

private inheritedAttributes: Attributes = {};

@Prop() labelText?: string;

componentWillLoad() {
  this.inheritedAttributes = inheritAttributes(this.el, ['aria-label']);
}

render() {
  return (
    <Host>
      <label>
        {this.labelText}
        <input type="checkbox" {...this.inheritedAttributes} />
      </label>
    </Host>
  )
}
```

----------------------------------------

TITLE: Initializing Ionic React Project with Capacitor
DESCRIPTION: Commands to initialize a new Ionic React project and enable Capacitor integration. These commands set up the basic project structure and prepare it for native app development.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/packages/react/README.md#2025-04-16_snippet_0

LANGUAGE: sh
CODE:
```
ionic init "My React App" --type=react
ionic integrations enable capacitor
```

----------------------------------------

TITLE: Using Ionic Overlay Controllers with Custom Elements
DESCRIPTION: Demonstrates how to use an overlay controller (modalController) with the custom elements build. This example shows the component definition and initialization required before creating and using a modal component.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/README.md#2025-04-16_snippet_3

LANGUAGE: typescript
CODE:
```
import { defineCustomElement } from '@ionic/core/components/ion-modal.js';
import { initialize, modalController } from '@ionic/core/components';

initialize();
defineCustomElement();

const showModal = async () => {
  const modal = await modalController.create({ ... });
  
  ...
}
```

----------------------------------------

TITLE: Ion Datetime Component API
DESCRIPTION: API documentation for the ion-datetime component, including date/time formatting options and presentation modes.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_12

LANGUAGE: typescript
CODE:
```
interface IonDatetimeProps {
  cancelText?: string;
  clearText?: string;
  color?: string;
  disabled?: boolean;
  presentation?: "date" | "date-time" | "month" | "month-year" | "time" | "time-date" | "year";
  locale?: string;
  mode?: "ios" | "md";
  value?: string | string[] | null;
}
```

----------------------------------------

TITLE: Defining the Ionic Segment View API
DESCRIPTION: Documentation for the ion-segment-view component that provides a container for segment content with scroll events and disabled state functionality.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_37

LANGUAGE: typescript
CODE:
```
ion-segment-view,shadow
ion-segment-view,prop,disabled,boolean,false,false,false
ion-segment-view,event,ionSegmentViewScroll,SegmentViewScrollEvent,true
```

----------------------------------------

TITLE: Opening Project in Native IDEs
DESCRIPTION: Command to open the project in platform-specific IDEs (Android Studio or Xcode) for building and emulating the native application.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/packages/react/README.md#2025-04-16_snippet_3

LANGUAGE: sh
CODE:
```
ionic capacitor open <android|ios>
```

----------------------------------------

TITLE: Defining ion-alert Component API
DESCRIPTION: API specification for the ion-alert component including props, methods, events, and CSS variables for both iOS and Material Design platforms.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_3

LANGUAGE: typescript
CODE:
```
ion-alert,scoped
ion-alert,prop,animated,boolean,true,false,false
ion-alert,prop,backdropDismiss,boolean,true,false,false
ion-alert,prop,buttons,(string | AlertButton)[],[],false,false
ion-alert,prop,cssClass,string | string[] | undefined,undefined,false,false
ion-alert,prop,enterAnimation,((baseEl: any, opts?: any) => Animation) | undefined,undefined,false,false
ion-alert,prop,header,string | undefined,undefined,false,false
ion-alert,prop,htmlAttributes,undefined | { [key: string]: any; },undefined,false,false
ion-alert,prop,inputs,AlertInput[],[],false,false
ion-alert,prop,isOpen,boolean,false,false,false
ion-alert,prop,keyboardClose,boolean,true,false,false
ion-alert,prop,leaveAnimation,((baseEl: any, opts?: any) => Animation) | undefined,undefined,false,false
ion-alert,prop,message,IonicSafeString | string | undefined,undefined,false,false
ion-alert,prop,mode,"ios" | "md",undefined,false,false
ion-alert,prop,subHeader,string | undefined,undefined,false,false
ion-alert,prop,translucent,boolean,false,false,false
ion-alert,prop,trigger,string | undefined,undefined,false,false
ion-alert,method,dismiss,dismiss(data?: any, role?: string) => Promise<boolean>
ion-alert,method,onDidDismiss,onDidDismiss<T = any>() => Promise<OverlayEventDetail<T>>
ion-alert,method,onWillDismiss,onWillDismiss<T = any>() => Promise<OverlayEventDetail<T>>
ion-alert,method,present,present() => Promise<void>
ion-alert,event,didDismiss,OverlayEventDetail<any>,true
ion-alert,event,didPresent,void,true
ion-alert,event,ionAlertDidDismiss,OverlayEventDetail<any>,true
ion-alert,event,ionAlertDidPresent,void,true
ion-alert,event,ionAlertWillDismiss,OverlayEventDetail<any>,true
ion-alert,event,ionAlertWillPresent,void,true
ion-alert,event,willDismiss,OverlayEventDetail<any>,true
ion-alert,event,willPresent,void,true
ion-alert,css-prop,--backdrop-opacity,ios
ion-alert,css-prop,--backdrop-opacity,md
ion-alert,css-prop,--background,ios
ion-alert,css-prop,--background,md
ion-alert,css-prop,--height,ios
ion-alert,css-prop,--height,md
ion-alert,css-prop,--max-height,ios
ion-alert,css-prop,--max-height,md
ion-alert,css-prop,--max-width,ios
ion-alert,css-prop,--max-width,md
ion-alert,css-prop,--min-height,ios
ion-alert,css-prop,--min-height,md
ion-alert,css-prop,--min-width,ios
ion-alert,css-prop,--min-width,md
ion-alert,css-prop,--width,ios
ion-alert,css-prop,--width,md
```

----------------------------------------

TITLE: Ion-Nav Navigation Component API
DESCRIPTION: Navigation component that manages a stack of pages/components. Provides methods for navigation operations like push, pop, and page management.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_29

LANGUAGE: typescript
CODE:
```
// Properties
swipeGesture: boolean;

// Methods
canGoBack(view?: ViewController): Promise<boolean>;
getActive(): Promise<ViewController | undefined>;
getByIndex(index: number): Promise<ViewController | undefined>;
getLength(): Promise<number>;
getPrevious(view?: ViewController): Promise<ViewController | undefined>;
insert<T extends NavComponent>(insertIndex: number, component: T, componentProps?: ComponentProps<T> | null, opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
insertPages(insertIndex: number, insertComponents: NavComponent[] | NavComponentWithProps[], opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
pop(opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
popTo(indexOrViewCtrl: number | ViewController, opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
popToRoot(opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
push<T extends NavComponent>(component: T, componentProps?: ComponentProps<T> | null, opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
removeIndex(startIndex: number, removeCount?: number, opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
setPages(views: NavComponent[] | NavComponentWithProps[], opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
setRoot<T extends NavComponent>(component: T, componentProps?: ComponentProps<T> | null, opts?: NavOptions | null, done?: TransitionDoneFn): Promise<boolean>;
```

----------------------------------------

TITLE: Modal Bottom Sheet Configuration
DESCRIPTION: Added bottom sheet functionality to modal components with properties for Angular. Allows configuration of breakpoints and handle display.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/CHANGELOG.md#2025-04-16_snippet_48

LANGUAGE: typescript
CODE:
```
interface ModalOptions {
  // Sheet modal properties
  breakpoints?: number[];
  backdropBreakpoint?: number;
  handle?: boolean;
}
```

----------------------------------------

TITLE: Ionic Split Pane Component Definition
DESCRIPTION: Configuration for ion-split-pane component with properties for controlling split view behavior and styling. Used for creating master-detail layouts.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_41

LANGUAGE: typescript
CODE:
```
interface IonSplitPane {
  contentId?: string;
  disabled: boolean;
  when: boolean | string;
  events: {
    ionSplitPaneVisible: { visible: boolean; }
  };
}
```

----------------------------------------

TITLE: Implementing Accessible Switch with VoiceOver Support in TSX
DESCRIPTION: TSX implementation for a switch component with VoiceOver support. Sets the required role and ARIA attributes for proper accessibility.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_22

LANGUAGE: tsx
CODE:
```
render() {
  const { checked, disabled } = this;

  return (
    <Host
      aria-checked={`${checked}`}
      aria-hidden={disabled ? 'true' : null}
      role="switch"
    >
      <input
        type="checkbox"
        role="switch"
      />
      ...
    </Host>
  );
}
```

----------------------------------------

TITLE: Defining ion-accordion Component API
DESCRIPTION: API specification for the ion-accordion component including props, parts, and shadow DOM encapsulation.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_0

LANGUAGE: typescript
CODE:
```
ion-accordion,shadow
ion-accordion,prop,disabled,boolean,false,false,false
ion-accordion,prop,mode,"ios" | "md",undefined,false,false
ion-accordion,prop,readonly,boolean,false,false,false
ion-accordion,prop,toggleIcon,string,chevronDown,false,false
ion-accordion,prop,toggleIconSlot,"end" | "start",'end',false,false
ion-accordion,prop,value,string,`ion-accordion-${accordionIds++}`,false,false
ion-accordion,part,content
ion-accordion,part,expanded
ion-accordion,part,header
```

----------------------------------------

TITLE: Configuring Angular Routes for Tabs
DESCRIPTION: Example of setting up Angular routes for a tabbed interface in Ionic 4. This replaces the previous ion-tab component with route configuration.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/BREAKING_ARCHIVE/v4.md#2025-04-16_snippet_40

LANGUAGE: typescript
CODE:
```
import { RouterModule, Routes } from '@angular/router';

import { TabsPage } from './tabs.page';

const routes: Routes = [
  {
    path: 'tabs',
    component: TabsPage,
    children: [
      {
        path: 'tab1',
        children: [
          {
            path: '',
            loadChildren: '../tab1/tab1.module#Tab1PageModule'
          }
        ]
      },
      {
        path: 'tab2',
        children: [
          {
            path: '',
            loadChildren: '../tab2/tab2.module#Tab2PageModule'
          }
        ]
      },
      {
        path: 'tab3',
        children: [
          {
            path: '',
            loadChildren: '../tab3/tab3.module#Tab3PageModule'
          }
        ]
      },
      {
        path: '',
        redirectTo: '/tabs/tab1',
        pathMatch: 'full'
      }
    ]
  },
  {
    path: '',
    redirectTo: '/tabs/tab1',
    pathMatch: 'full'
  }
];
```

----------------------------------------

TITLE: Using toHaveReceivedEventDetail Matcher in Ionic Playwright Tests
DESCRIPTION: Demonstrates the toHaveReceivedEventDetail custom matcher, which verifies that an event has been fired with a specific event detail payload. This example checks the ionChange event detail after typing in an input.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/core/testing/api.md#2025-04-16_snippet_7

LANGUAGE: typescript
CODE:
```
import { configs, test } from '@utils/test/playwright';

configs().forEach(({ config, screenshot, title }) => {
  test.describe(title('my test block'), () => {
    test('my custom test', ({ page }) => {
      await page.setContent(`
        <ion-input label="Email"></ion-input>
      `, config);
      
      const ionChange = await page.spyOnEvent('ionChange');
      const input = page.locator('ion-input');
      
      await input.type('hi@ionic.io');
  
      await ionChange.next();
      await expect(ionChange).toHaveReceivedEventDetail({ value: 'hi@ionic.io' });
    });
  });
});
```

----------------------------------------

TITLE: RTL-Aware Styling in Ionic Components (CSS)
DESCRIPTION: This snippet shows how to implement RTL-aware styling in Ionic components. It demonstrates the use of mixins for automatic RTL handling and provides a workaround for WebKit compatibility issues in shadow components.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_27

LANGUAGE: css
CODE:
```
@include transform-origin(start, center);
```

LANGUAGE: css
CODE:
```
:host {
  transform-origin: left center;
}

:host(.my-cmp-rtl) {
  transform-origin: right center;
}
```

----------------------------------------

TITLE: Defining the Ionic Select API
DESCRIPTION: Documentation for the ion-select component which provides a customizable dropdown selection interface with various display options including alert, action sheet, popover, and modal interfaces.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_38

LANGUAGE: typescript
CODE:
```
ion-select,shadow
ion-select,prop,cancelText,string,'Cancel',false,false
ion-select,prop,color,"danger" | "dark" | "light" | "medium" | "primary" | "secondary" | "success" | "tertiary" | "warning" | string & Record<never, never> | undefined,undefined,false,true
ion-select,prop,compareWith,((currentValue: any, compareValue: any) => boolean) | null | string | undefined,undefined,false,false
ion-select,prop,disabled,boolean,false,false,false
ion-select,prop,errorText,string | undefined,undefined,false,false
ion-select,prop,expandedIcon,string | undefined,undefined,false,false
ion-select,prop,fill,"outline" | "solid" | undefined,undefined,false,false
ion-select,prop,helperText,string | undefined,undefined,false,false
ion-select,prop,interface,"action-sheet" | "alert" | "modal" | "popover",'alert',false,false
ion-select,prop,interfaceOptions,any,{},false,false
ion-select,prop,justify,"end" | "space-between" | "start" | undefined,undefined,false,false
ion-select,prop,label,string | undefined,undefined,false,false
ion-select,prop,labelPlacement,"end" | "fixed" | "floating" | "stacked" | "start" | undefined,'start',false,false
ion-select,prop,mode,"ios" | "md",undefined,false,false
ion-select,prop,multiple,boolean,false,false,false
ion-select,prop,name,string,this.inputId,false,false
ion-select,prop,okText,string,'OK',false,false
ion-select,prop,placeholder,string | undefined,undefined,false,false
ion-select,prop,required,boolean,false,false,false
ion-select,prop,selectedText,null | string | undefined,undefined,false,false
ion-select,prop,shape,"round" | undefined,undefined,false,false
ion-select,prop,toggleIcon,string | undefined,undefined,false,false
ion-select,prop,value,any,undefined,false,false
ion-select,method,open,open(event?: UIEvent) => Promise<any>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maelzin13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
