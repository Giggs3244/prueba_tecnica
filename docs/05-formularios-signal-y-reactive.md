# 05 — Formularios: Signal Forms y Reactive Forms

Fecha: 2026-10-13 · Complemento de la fase 4

## Decisión

La aplicación usa **solo Signal Forms**, el sistema de formularios recomendado desde Angular 22.

Se llegó a construir y probar una variante con Reactive Forms de la misma página, pero se retiró del proyecto para no mantener dos implementaciones. Este documento conserva esa variante como material de referencia: es probable que el banco trabaje con una versión de Angular anterior a la 22, donde Signal Forms no existe, y que pregunten por Reactive Forms en la entrevista.

Nada del código de la sección "Variante con Reactive Forms" existe hoy en el repositorio.

## Los dos sistemas lado a lado

| | Signal Forms (lo que usa la app) | Reactive Forms |
|---|---|---|
| Declaración | `model = signal({ userId: '', password: '', rememberUser: false })` y `demoForm = form(this.model)` | `demoForm = inject(NonNullableFormBuilder).group({ userId: '', password: '', rememberUser: false })` |
| Enlace en plantilla | `[formField]="demoForm.userId"` | `[formGroup]="demoForm"` en el contenedor y `formControlName="userId"` en cada control |
| Leer el valor | `model()` — ya es un signal | `demoForm.getRawValue()`, o `toSignal(demoForm.valueChanges)` para usarlo en la plantilla |
| Imports del componente | `FormField` | `ReactiveFormsModule` |
| Validación | Reglas en un esquema: `form(model, (path) => { required(path.userId); })` | `Validators.required` en cada control |
| Estado de un campo | `demoForm.userId().touched()`, `.valid()`, `.dirty()` (signals) | `demoForm.controls.userId.touched`, `.valid`, `.dirty` (propiedades) |
| Reaccionar a cambios | `computed()` o `effect()` sobre el signal | Suscripción a `valueChanges` |
| Control propio | El componente expone un `model()` llamado `value` o `checked` | El componente (o una directiva) implementa `ControlValueAccessor` |
| Disponible desde | Angular 22 (estable) | Angular 2 |

## Signal Forms en la app

Así está escrita hoy la página de prueba `home.page.ts`:

```ts
protected readonly model = signal({ userId: '', password: '', rememberUser: false });
protected readonly demoForm = form(this.model);
```

```html
<app-input placeholder="Documento de identidad" [formField]="demoForm.userId" />
<app-input placeholder="Contraseña" type="password" [formField]="demoForm.password" />
<app-toggle [formField]="demoForm.rememberUser">Recuerda mi usuario</app-toggle>
```

`app-input` y `app-toggle` funcionan con `[formField]` porque cumplen los contratos `FormValueControl<string>` y `FormCheckboxControl`: tienen un `model()` llamado `value` o `checked`, una entrada `disabled` y una salida `touch`.

## Variante con Reactive Forms (referencia, no está en el proyecto)

### Plantilla de la página

Es la misma plantilla de la galería, con dos diferencias: el contenedor lleva `[formGroup]` y cada control lleva `formControlName` en lugar de `[formField]`.

```html
<app-header title="Componentes" leading="menu" />
<app-header title="Detalle de producto" leading="back" />

<ion-content>
  <main class="showcase" [formGroup]="demoForm">
    <app-input
      placeholder="Documento de identidad"
      icon="assets/icons/user-input-gray.svg"
      inputMode="numeric"
      autocomplete="username"
      formControlName="userId"
    />
    <app-input
      placeholder="Contraseña"
      icon="assets/icons/key-input-gray.svg"
      type="password"
      autocomplete="current-password"
      formControlName="password"
    />
    <app-toggle formControlName="rememberUser">Recuerda mi usuario</app-toggle>
    <app-button (click)="showLoading()">INICIAR SESIÓN</app-button>
    <app-button variant="outline">CERRAR SESIÓN</app-button>

    <output class="showcase__state">{{ model().userId }} · {{ model().password?.length }} · {{ model().rememberUser }}</output>

    @for (product of products; track product.number) {
      <app-product-card [product]="product" />
    }
    <app-product-card-skeleton />

    <app-placeholder-content
      title="Pantalla
de transacciones"
      description="Esta pantalla es únicamente para validar como se construye la navegación de la aplicación, por lo tanto, este puede ser el único contenido"
    />
  </main>
</ion-content>

@if (loading()) {
  <app-loading-overlay />
}
```

### Clase de la página

```ts
@Component({
  selector: 'app-home',
  templateUrl: 'home.page.reactive.html',
  styleUrls: ['home.page.scss'],
  imports: [
    IonContent,
    ReactiveFormsModule,
    ButtonComponent,
    HeaderComponent,
    InputComponent,
    InputValueAccessorDirective,
    LoadingOverlayComponent,
    PlaceholderContentComponent,
    ProductCardComponent,
    ProductCardSkeletonComponent,
    ToggleComponent,
    ToggleValueAccessorDirective,
  ],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class HomeReactivePage {
  protected readonly demoForm = inject(NonNullableFormBuilder).group({
    userId: '',
    password: '',
    rememberUser: false,
  });
  protected readonly model = toSignal(this.demoForm.valueChanges, { initialValue: this.demoForm.getRawValue() });
  protected readonly loading = signal(false);
}
```

Puntos a notar:

- `NonNullableFormBuilder` hace que los controles nunca valgan `null` al reiniciarse; el tipo del formulario queda `string` y `boolean`.
- `valueChanges` es un observable. Con detección de cambios OnPush y sin Zone.js, la plantilla no se entera de un observable; por eso se convierte con `toSignal`.
- El valor que emite `valueChanges` tiene todas las propiedades opcionales (un control deshabilitado no aparece), de ahí el `?.` en `model().password?.length`.

### El puente: `ControlValueAccessor`

`formControlName` no sabe hablar con un componente propio. Hace falta un accesor de valor. Se resolvió con una directiva por componente, que se activa sola cuando el elemento lleva `formControlName` o `formControl`:

```ts
@Directive({
  selector: 'app-input[formControlName], app-input[formControl]',
  providers: [
    { provide: NG_VALUE_ACCESSOR, useExisting: forwardRef(() => InputValueAccessorDirective), multi: true },
  ],
})
export class InputValueAccessorDirective implements ControlValueAccessor {
  private readonly host = inject(InputComponent, { self: true });
  private notifyChange: (value: string) => void = () => undefined;
  private notifyTouched: () => void = () => undefined;
  private isWriting = false;

  constructor() {
    this.host.value.subscribe((value) => {
      if (!this.isWriting) {
        this.notifyChange(value);
      }
    });
    this.host.touch.subscribe(() => this.notifyTouched());
  }

  writeValue(value: string | null): void {
    this.isWriting = true;
    this.host.value.set(value ?? '');
    this.isWriting = false;
  }

  registerOnChange(notifyChange: (value: string) => void): void {
    this.notifyChange = notifyChange;
  }

  registerOnTouched(notifyTouched: () => void): void {
    this.notifyTouched = notifyTouched;
  }

  setDisabledState(isDisabled: boolean): void {
    this.host.disabledByForm.set(isDisabled);
  }
}
```

La directiva del toggle era igual, cambiando `value` por `checked` y `string` por `boolean`.

Para que `setDisabledState` funcionara, el componente necesitaba un signal adicional, porque una entrada (`input()`) no se puede escribir desde fuera de la plantilla:

```ts
readonly disabledByForm = signal(false);
protected readonly isDisabled = computed(() => this.disabled() || this.disabledByForm());
```

### Qué más hacía falta para intercambiar las variantes

- Una opción de configuración (`formsEngine: 'signal' | 'reactive'`) en los archivos de entorno.
- La ruta elegía qué página cargar según esa opción, cada una en su propio archivo de carga diferida.
- Una excepción en la regla de lint que exige el prefijo `app` en selectores de directiva, porque el selector se engancha a `formControlName`.

## Por qué se descartó mantener las dos

| Motivo | Detalle |
|---|---|
| Doble mantenimiento | Cada formulario tendría dos clases, dos plantillas y dos juegos de pruebas |
| Código que no se ejecuta | La variante inactiva no aporta al usuario final |
| Superficie extra en los componentes | `disabledByForm` y las directivas existían solo para el sistema que no se usa |
| El proyecto es Angular 22 | Signal Forms es estable y es la recomendación actual |

Si hubiera que llevar la app a una versión de Angular sin Signal Forms, el camino es el descrito arriba: agregar las directivas de accesor, cambiar `[formField]` por `formControlName` y declarar el formulario con `FormBuilder`.

## Conceptos

### `ControlValueAccessor`

Es el contrato que Reactive Forms exige a cualquier control. Tiene cuatro métodos:

| Método | Quién lo llama | Para qué |
|---|---|---|
| `writeValue(valor)` | El formulario | Mostrar en el control un valor puesto desde código |
| `registerOnChange(fn)` | El formulario | Entrega la función que el control debe llamar cuando el usuario cambia el valor |
| `registerOnTouched(fn)` | El formulario | Entrega la función que el control debe llamar al perder el foco |
| `setDisabledState(bool)` | El formulario | Habilitar o deshabilitar el control |

El accesor se registra con el token `NG_VALUE_ACCESSOR`. Sin él, `formControlName` sobre un componente propio falla con "No value accessor".

### El problema del eco

Cuando el formulario escribe un valor (`writeValue`), el componente actualiza su `model()` y eso dispara su evento de cambio. Si el accesor reenviara ese evento al formulario, el control quedaría marcado como modificado por el usuario (`dirty`) sin que nadie lo tocara. Por eso el accesor levanta una bandera (`isWriting`) mientras escribe y descarta los cambios que ocurren en ese momento.

### Por qué una directiva puede hablar con el componente

`inject(InputComponent, { self: true })` pide el componente que está en el mismo elemento que la directiva. Con esa referencia lee y escribe su `model()` y se suscribe a su salida `touch`.

### Accesor en directiva o dentro del componente

Lo habitual en proyectos con Reactive Forms es que el propio componente implemente `ControlValueAccessor`. Aquí se puso en una directiva aparte para que el componente siguiera siendo un control de Signal Forms sin mezclar los dos contratos.

### `FormGroup`, `FormControl` y `FormBuilder`

- `FormControl` representa un campo: guarda su valor, su validez y si fue tocado o modificado.
- `FormGroup` agrupa controles por nombre y calcula el estado conjunto.
- `FormBuilder` es un atajo para crear grupos y controles con menos código.

### Estados de un control

| Estado | Significado |
|---|---|
| `pristine` / `dirty` | El usuario no ha cambiado / ha cambiado el valor |
| `untouched` / `touched` | El control no ha perdido / ha perdido el foco alguna vez |
| `valid` / `invalid` | Pasa / no pasa sus validadores |
| `disabled` | Excluido del valor y de la validación del grupo |

Angular refleja estos estados como clases CSS en el elemento (`ng-dirty`, `ng-touched`, `ng-invalid`).

### `toSignal`

Convierte un `Observable` en un signal. Sirve para leer `valueChanges` desde una plantilla sin el pipe `async`.

### Signal Forms en una frase

El estado del formulario es un signal con los datos; `form()` construye encima un árbol de campos con su validez, estado "tocado" y deshabilitado, todo legible como signals. No hay `FormGroup` ni suscripciones.

## Qué se comprobó antes de retirar la variante

La variante reactiva se probó antes de eliminarla:

- Valor inicial y valores puestos desde código se muestran sin marcar el control como `dirty`.
- Lo que escribe el usuario llega al control y lo marca `dirty`.
- Volver a escribir un valor que antes se puso desde código se registra bien.
- Perder el foco marca `touched`.
- Deshabilitar y habilitar el control se refleja en el componente.
- La misma batería de pruebas de la página pasaba con las dos variantes, y en el navegador el formulario se actualizaba al escribir y al activar el toggle.

## Estado del proyecto tras retirarla

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 95 / 95 |
| `ng lint` | Sin errores |
