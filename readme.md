- @stencil/core 3.4.2 and 4.2.0
- @stencil/angular-output-target: ^0.8.1

Building a custom-component which uses @ionic/core to render some components:

1. In stencil.config.ts I've defined `namespace: 'CustomComonentNs'` but anyway the `components.d.ts` generates:
```ts
export namespace Components {
[...]
}
```

If this is fixed, @stencil/angular-output-target should also import and generate components.ts from corrected namespace.

2. The generated component.ts includes **all** components from @ionic/core, it does not filter the components really used. In example, the `custom-component` only uses `ion-content` but in generated `angular-workspace/projects/component-library/src/lib/stencil-generated/component.ts` we can find:

```ts
import { Components } from 'custom-component';

[...]

@ProxyCmp({
  inputs: ['disabled', 'readonly', 'toggleIcon', 'toggleIconSlot', 'value']
})
@Component({
  selector: 'ion-accordion',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<ng-content></ng-content>',
  // eslint-disable-next-line @angular-eslint/no-inputs-metadata-property
  inputs: ['disabled', 'readonly', 'toggleIcon', 'toggleIconSlot', 'value'],
})
export class IonAccordion {
  protected el: HTMLElement;
  constructor(c: ChangeDetectorRef, r: ElementRef, protected z: NgZone) {
    c.detach();
    this.el = r.nativeElement;
  }
}

export declare interface IonAccordion extends Components.IonAccordion {}

[...]
```

3. It also imports `Components` from components.d.ts, but as stated in https://github.com/ionic-team/stencil-ds-output-targets/issues/232, we should declare all the components in `interface.d.ts`. This is quite difficult considering that `components.ts` includes all the components from `@ionic/core`, so it makes really difficult to maintain. Would be easier if output-target generator would exclude unused components. Anyway I imagine that the problem is with any external library or web component that would define it's custom-elements, they should be re-exported as described by @sean-perkins in mentioned issue.

If the step 3 is not done, angular library will not compile, complaining about:

```
projects/component-library/src/lib/stencil-generated/components.ts:278:58 - error TS2694: Namespace '"/Volumes/projects/stencil-library/dist/types/components".Components' has no exported member 'IonAccordion'.

278 export declare interface IonAccordion extends Components.IonAccordion {}
```
