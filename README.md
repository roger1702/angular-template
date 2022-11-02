# PickUp Experience

![N|Solid](https://static.construible.es/media/2021/11/cemex-logo-dest.png)

Proyecto basado en Angular 14.

## Instalación

```sh
npm install o npm i
```

o si se presenta un error durante la instalación

```sh
npm install --legacy-peer-deps
```

Nota: Para los paquetes de Cemex design se debe crear el archivo .npmrc a nivel
raíz y agregar la configuración correspondiente. Para la configuración debe
solicitarse al equipo Neoris correspondiente

## Ejecución

```sh
ng serve o npm start
```

### Generar archivos para QA

```sh
ng build --configuration=qa o npm run build:qa
```

### Generar archivos para producción

```sh
ng build --configuration=prod o npm run build:prod
```

## Paquetes

- [Cemex design](https://cemex.design/components) - Bibilioteca de estilos para
  proyectos cemex
- [Animate css](https://animate.style/) - Animaciones CSS
- [ngx translate](https://github.com/ngx-translate/core) - paquete para
  internacionalización.
- [Bootstrap](https://getbootstrap.com/docs/5.1/getting-started/introduction/) -
  Contiene cargado solo el sistema de rejillas
- [commitizen](https://github.com/commitizen/cz-cli) - Paquete para estandarizar
  commits en git
- [husky](https://typicode.github.io/husky/#/) - Paquete para preparar los
  commits
- [ngx-scanner-qrcode](https://www.npmjs.com/package/ngx-scanner-qrcode) -
  Paquete para scanner QR

### Guía de desarrollo

#### CSS

##### Grid

Para el sistema de regillas usar las regillas de bootstrap. El proyecto
actualmente tiene cargado solo el sistema de regillas

```sh
 <div class="h-100 d-flex
             flex-row justify-content-between
             align-items-center">
</div>
```

o

```sh
  <div class="container">
   <div class="row">
     <div class="col">col</div>
   </div>
 </div>
```

- [Generador CSS Flexbox boostrap](https://mdbootstrap.com/docs/standard/tools/builders/flexbox/)

##### Custom CSS

Para el uso de clases personalizadas, deben estar bajo la métodología BEM

- [Ejemplo de uso](https://fixu.cl/que-es-la-metodologia-bem-y-como-se-aplica-al-front-end/)
- [Ejemplo de uso](https://www.youtube.com/watch?v=YaAkV--25fg)

##### Medidas

Las medidas en CSS deben ser relativas y no absolutas, así que deben estar en
rem o % y no en px

- [Convertidor px to rem](https://nekocalc.com/es/px-a-rem-conversor)

##### Especificidad

Evitar colocar en las tags el atributo style ya que esto genera problemas de
especificidad. De igual forma evitar en CSS colocar la opción !important

- [Ejemplo](https://codigofacilito.com/articulos/especificidad-css)

##### Media querys

El proyecto cuenta con mixin para solventar el problema responsivo. En caso de
requerirse otros medidas se puede agregar al archivo device-size.scss

Nota: El archivo style.scss debe estar lo más limpio posible, en caso de
requerir agregar estilos en general, crear un archivo dentro de la carpeta
assets -> sass.

#### Estructura

La estructura del proyecto se encuentra relacionado ruta vs carpeta para que de
esta forma sea más sencilla su navegación. Por lo cual dentro de cada menu
principal se pueden agregar sub carpetas para cada sub ruta. En caso de uso de
modales se debe crear la carpeta modal al nivel requerido.

El proyecto Angular esta bajo el patrón atomico. Por lo cual cada ruta, sub ruta
o modal usado debe contener su propio modulo correspondiente.

- [Más información patrón atomico](https://medium.com/weekly-webtips/angular-clean-arquitecture-d40e9c50f51)

#### Componentes

##### Variables en componentes

Para el orden de la declaración de variables, se debe tener el siguiente orden

- variables con decorador
- variables privadas
- variables publicas

Ejemplo

```sh
 @Input() name:string;
 @ViewChild('someInput') someInput: ElementRef;
 private _age:number;
 private _formData:FormData;
 dataTable: IExample[];
```

##### Variables y métodos privados

Para las variables y métodos privados deben ir con un \_ para su
identitificación

Ejemplo

```sh
 private _age:number;

 private _getName():void {
     ...
 }
```

##### Orden de los métodos

Para los métodos, se debe llevar acabo en el siguiente orden

- Constructor
- Métodos del ciclo de vida del componente
- Métodos privados
- Métodos publicos

Ejemplo

```sh
 constructor():void{}

 ngOnchanges():void{}
 ngOnInit():void{}
 ngAfterViewInit():void{}
 ngOnDestroy():void{}

 private _getName():void{}

 setNameUser():void{}
```

##### Tipo de retorno en métodos

Todos los métodos deben especificar si retornan algun tipo o son de tipo void

##### Finalizar la subscripción con Observables.

Con el objetivo de evitar fugas de memoria por el uso de observables, debe
implementarse el uso de las siguientes técnicas.

- Usar el pipe | async para no subscribirse del lado del componente
- Usar el operador take(number)
- Usar el operador takeUntil(notifier: Observable)
- Usar la clase Subscription para crear una nueva instancia

Ejemplo pipe async

```sh
<span *ngFor="let item of itemUsers$ |async">
</span>
```

Componente

```sh
itemUsers$:Observable<Users[]>
constructor(private _userServices:UserServices){}

ngOnInit(){
    this._getUsers();
}
private _getUsers():void {
    this.itemUsers$ = this._userServices.getUser();
}
```

Ejemplo uso de take

```sh
constructor(private _userServices:UserServices){}

ngOnInit(){
    this._getUsers();
}
private _getUsers():void {
   this._userServices.getUser().pipe(take(1)).subscribe((response:Users) =>{
     ...
     // take(1) solo tomara la primer emisión, posteriormente cerrara la subscripción
   });
}
```

Ejemplo uso de takeUntil

```sh
private readonly _UnSubscribe$:Subject<void> = new Subject<void>();
constructor(private _userServices:UserServices){}

ngOnInit(){
    this._getUsers();
}
ngOnDestroy(){
    this._UnSubscribe$.next();
    this._UnSubscribe$.complete();
}
private _getUsers():void {
   this._userServices.getUser()
        .pipe(takeUntil(this._UnSubscribe$))
        .subscribe((response:Users) =>{
     ...
   });
}
```

Ejemplo uso de Subject

```sh
private readonly subscription$ = new Subscription();
constructor(private _userServices:UserServices){}

ngOnInit(){
    this._getUsers();
}
ngOnDestroy(){
    this._subscription$.unsubscribe();
}
private _getUsers():void {
    this._subscription$.add(
        this._userServices.getUser()
        .subscribe((response:Users) =>{
            ...
        });
    );
}
```

Nota: Todos los observables como estandar deben llevar al final el simbolo de $

- [Más información](https://kgotgit.medium.com/rxjs-subject-subscription-heap-memory-analysis-909dc173a613)

##### Detección de cambios

Cada vez que se pasa información a componentes hijos, se debe aplicar una
estrategía de detección de cambios. Para ello se debe implementar
ChangeDetectorRef

Ejemplo

```sh
import {  Component, ChangeDetectorRef } from '@angular/core';

dataGrid:Users[];

export class AppComponent {
constructor(private _userServices:UserServices,
            private _cd:ChangeDetectorRef ){}

ngOnInit(){
    this._getUsers();
}
private _getUsers():void {
    this._userServices.getUser()
        .subscribe((response:Users) =>{
            this.dataUsers = response;
            this._cd.detectChanges(); //Se ejecuta detección de cambios
        });
}
}

```

HTML

```sh
<app-custom [data]="dataGrid"></app-custom>
```

## Desarrollado por

Team Neoris
