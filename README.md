# TeSirvoApp

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.6.0.

## Servicio UploadFiles

```JavaScript
import { HttpClient, HttpEvent, HttpRequest } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { GLOBAL } from './global';

@Injectable({
  providedIn: 'root'
})
export class UploadFilesService {
  public url: String
  constructor(private _http: HttpClient){
    this.url = GLOBAL.url;
  }
  //Metodo que envia los archivos al endpoint /upload 
  upload(file: File): Observable<HttpEvent<any>>{
    const formData: FormData = new FormData();
    formData.append('files', file);
   
    const req = new HttpRequest('POST', `${this.url}file/upload`, formData, {
      reportProgress: true,
      responseType: 'json'
    });
    return this._http.request(req);
  }

  //Metodo para Obtener los archivos
  getFiles(){
    return this._http.get(`${this.url}file/files`);
  }

  //Metodo para borrar los archivos
  deleteFile(filename: string){
    return this._http.get(`${this.url}delete/${filename}`);
  }
}
```

## NavBar Vista - Template HTML
```HTML
<nav class="navbar navbar-expand-md navbar-dark bg-dark bg-faded">
    <div class="container-fluid">
        <a class="navbar-brand" href="#">
            <img src="assets/img/logo.png" alt="LOGO UNAB" width="30" height="24">
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item">
                    <a class="nav-link active" aria-current="page" routerLink="home" routerLinkActive="active" href="#">Home</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" routerLink="registro" routerLinkActive="active">Registrar</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" routerLink="usuarios" routerLinkActive="active">Administrador de Usuarios</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" routerLink="about" routerLinkActive="active">About</a>
                </li>
            </ul>
            <form class="d-flex" role="search">
                <input class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
                <button class="btn btn-outline-success" type="submit">Search</button>
            </form>
        </div>
    </div>
</nav>
```

## HTML de About

```HTML
<div class="jumbotron jumbotron-fluid animated fadeIn fast">

    <div class="container">
        <h1 class="display-3">Te Sirvo App</h1>
        <p class="lead">Esta es una aplicación donde puedes consultar servicios ofertados.</p>
    </div>

</div>
```

## Las Rutas

```JavaScript
const routes: Routes = [
  { path:'home',  component:HomeComponent},
  { path:'registro',  component:RegisterComponent},
  { path:'usuarios',  component:UsuariosComponent},
  { path:'usuarios/form',  component:UsuarioFormComponent},
  { path:'usuarios/form/:id',  component:UsuarioFormComponent},
  { path:'about',  component:AboutComponent},
  { path:'**', pathMatch:'full', redirectTo:'home'},
];
```
## El Controllador form.component.ts

``` JavaScript
import { HttpEventType, HttpResponse } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { Observable } from 'rxjs';
import { Usuario } from 'src/app/models/usuario';
import { UploadFilesService } from 'src/app/services/upload-files.service';
import { UsuarioService } from 'src/app/services/usuario.service';
import swal from 'sweetalert2';

@Component({
  selector: 'app-usuario-form',
  templateUrl: './usuario-form.component.html',
  styles: [
  ]
})
export class UsuarioFormComponent implements OnInit {

  public usuario:Usuario = new Usuario('','','','','','',null);
  public titulo:string="Crear Usuario";
  public mensaje:string="";
  public files:any;
  message="";
  fileName="";
  clave_encriptada="";
  fileInfos:Observable<any> | undefined;

  constructor(
    private usuarioService:UsuarioService,
    private uploadFilesService:UploadFilesService,
    private router:Router,
    private activatedRoute:ActivatedRoute) { }

  ngOnInit(): void {
    this.cargarUsuario();
  }

  cargarUsuario(): void{
    this.activatedRoute.params.subscribe(params => {
      let id = params['id']
      console.log(params)
      if(id){
        this.usuarioService.getUsuario(id).subscribe( (usuario) => this.usuario = usuario)
        this.clave_encriptada = this.usuario.password;
      }
    })
  }
  selectFiles(event:any){
    event.target.files.length==1 ? this.fileName=event.target.files[0].name:this.fileName=event.target.files.length + "archivos";
    this.files = event.target.files
  }
  uploadFiles(){
    this.message="";
    this.upload(0,this.files[0]);
   }

   upload(index:number, file:any) {
    //this.progressInfo[index] = { value: 0, fileName: file.name };

    this.uploadFilesService.upload(file).subscribe(
      event => {
        if (event.type === HttpEventType.UploadProgress) {
          //this.progressInfo[index].value = Math.round(100 * event.loaded / event.total);
        } else if (event instanceof HttpResponse) {
          this.usuario.urlFoto=this.fileName;
          this.fileInfos = this.uploadFilesService.getFiles();
          this.fileInfos.subscribe({
            next: (data) => {
              if(data){
                console.log(data)
                 }
              }
          });
        }
      },
      err => {
        //this.progressInfo[index].value = 0;
        this.message = 'No se puede subir el archivo ' + file.name;
      });
  }
  create(formRegister:any):void{
    console.log(this.usuario);
    this.usuarioService.register(this.usuario).subscribe({
   next: (data) => {
    if(data){
      console.log(data)
      this.mensaje ="El Registro se ha realizado correctametne...";
      this.usuario = new Usuario('','','','','','',null);
      swal.fire("Usuario Registrado",this.mensaje,'success');
      formRegister.reset();
      this.router.navigate(['../usuarios'])
     }else{
      this.mensaje = "Error al Registrar Usuario";
     }
   },
   error: (error)=>{
    console.log(<any>error);
   },
   complete: ()=>{console.info('Proceso Terminado')}
   });
   }

   update():void{
    this.usuarioService.update(this.usuario).subscribe( usuario => {
      this.router.navigate(['../usuarios'])
      swal.fire('Usuario Actualizado', `Usuario ${usuario.nombreCompleto} actualizado con éxito!`, 'success')
    }
    )
  }

}
```
## El Formulario de Usuarios
```HTML
<div class="card bg-dark text-white">
    <div class="card-header">{{ titulo }}</div>
    <div class="card-body">

        <form #formRegister="ngForm">
            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Nombre Completo</label>
                <div class="col-sm-6">
                    <input type="text " class="form-control" name="nombre" #nombre="ngModel" [(ngModel)]="usuario.nombreCompleto" placeholder="Nombre Competo" ngModel required>
                    <span *ngIf="!nombre.valid && nombre.touched">El Nombre Es Obligatorio</span>
                </div>
            </div>

            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Profesión</label>
                <div class="col-sm-6">
                    <input type="text" name="profesion" #profesion="ngModel" [(ngModel)]="usuario.profesion" placeholder="Profesion" ngModel required>
                    <span *ngIf="!profesion.valid && profesion.touched">La Profesion Es Obligatoria</span>
                </div>
            </div>

            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Descripción del Servicio</label>
                <textarea type="text" name="descripcionServicio" #descripcionServicio="ngModel" [(ngModel)]="usuario.descripcionServicio" placeholder="Descripcion Servicio" ngModel required></textarea>
                <span *ngIf="!descripcionServicio.valid && descripcionServicio.touched">La Profesion Es Obligatoria</span>
            </div>

            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Usuario</label>
                <input type="text" name="username" #username="ngModel" [(ngModel)]="usuario.username" placeholder="Usuario" ngModel required>
                <span *ngIf="!username.valid && username.touched">El Usuario Es Obligatorio</span>
            </div>

            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Contraseña</label>
                <input type="password" name="password" #password="ngModel" [(ngModel)]="usuario.password" [disabled]="clave_encriptada" placeholder="Password " ngModel required>
                <span *ngIf="!password.valid && password.touched">Debe Digitar una Contraseña</span>
            </div>
            <div class="form-group row">
                <label for="nombre" class="col-form-label col-sm-2">Foto</label>
                <div class="col-sm-6">
                    <button class="btn btn-info" id="selectButton" mat-raised-button (click)="fileInput.click()">Seleccione Imagen</button>
                    <input #fileInput type="file" hidden multiple (change)="selectFiles($event)" />
                    <span *ngIf="fileName">{{fileName}}</span>
                    <button class="btn btn-info" id="uploadButton" mat-raised-button *ngIf="fileName" [disabled]="!fileName" (click)="uploadFiles()">
                   Subir Archivos
                  </button>
                </div>
            </div>

            <div class="form-group row">
                <div class="col-sm-6">
                    <button class="btn btn-primary" role="button" (click)='create(formRegister)' [disabled]="!formRegister.form.valid || !usuario.urlFoto" *ngIf="!usuario.fechaCreacion else elseBlock">Crear</button>
                    <ng-template #elseBlock>
                        <button class="btn btn-primary" role="button" (click)='update()' [disabled]="!formRegister.form.valid || !usuario.urlFoto">Editar</button>
                    </ng-template>
                </div>
            </div>


        </form>

    </div>
</div>
```
 
## Controlador Usuarios - Tabla

```JavaScript
import { Component, OnInit } from '@angular/core';
import { Usuario } from '../../models/usuario'
import { UsuarioService } from '../../services/usuario.service';
import swal from 'sweetalert2'

@Component({
  selector: 'app-usuarios',
  templateUrl: './usuarios.component.html',
  styles: [
  ]
})
export class UsuariosComponent implements OnInit {
  public usuarios:Usuario[]=[];

  constructor(private _usuarioService: UsuarioService) { }

  ngOnInit(): void {
    //if(!this._usuarioService.esAutenticado()){
    //  this._router.navigate(['login']);
    //}else{
      this._usuarioService.listasUsuarios().subscribe(data => (this.usuarios=data));
    //}
  }

  eliminar(usuario: Usuario): void {
     
    swal.fire({
      title: 'Está seguro?',
      text:`¿Seguro que desea eliminar al cliente ${usuario.nombreCompleto} ?`,
      showCancelButton: true,
      confirmButtonColor: '#3085d6',
      cancelButtonColor: '#d33',
      confirmButtonText: 'Si, eliminar!',
      cancelButtonText: 'No, cancelar!',
      //icon: 'warning',
      reverseButtons: true
    }).then((result:any) => {
      if (result.value) { 
        this._usuarioService.delete(usuario.username).subscribe(
          response => {
            this.usuarios = this.usuarios.filter(cli => cli !== usuario)
            swal.fire(
              'Cliente Eliminado!',
              `Cliente ${usuario.nombreCompleto} eliminado con éxito.`,
              'success'
            )
          }
        )

      }
    })
  }

}
```
## Vista HTML de Usuarios - Tabla
```HTML
<div class="card border-primary mb-3">
    <div class="card-header">Perfil de Usuario</div>
    <div class="card-body text-primary">
        <h5 class="card-title">Listado de Usuarios</h5>
        <div class="my-2 text-left">
            <button class="btn btn-rounded btn-primary" type="button" [routerLink]="['../usuarios/form']">
          Registrar  Usuario 
        </button>
        </div>
        <table id="filas" class="table table-bordered table-striped">
            <thead>
                <tr>
                    <th>Nombre Usuario</th>
                    <th>Nombre Competo</th>
                    <th>Profesion</th>
                    <th>Editar</th>
                    <th>Eliminar</th>
                </tr>
            </thead>
            <tbody>
                <tr *ngFor="let usuario of usuarios">
                    <td>{{usuario.username}}</td>
                    <td> {{usuario.nombreCompleto}} </td>
                    <td> {{usuario.profesion}} </td>
                    <td>
                        <button type="button" class="btn btn-success" [routerLink]="['../usuarios/form', usuario.username]">Editar</button>
                        <!-- <a [routerLink]="['../editar-partido', usuario.username]">Editar </a><a [routerLink]="['../editar-partido', usuario.username]"> - Eliminar </a>-->
                    </td>
                    <td>
                        <button type="button" class="btn btn-danger" (click)="eliminar(usuario)">Eliminar</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
```
## Controlador Componente Tarjeta de Usuario (usuario-tarjeta)

usuario-tarjeta.component.ts:

```JavaScript
import { Component, OnInit } from '@angular/core';
import { Input, Output, EventEmitter } from '@angular/core';
import { Router } from '@angular/router';  
import { GLOBAL } from '../../../services/global';
@Component({
  selector: 'app-usuario-tarjeta',
  templateUrl: './usuario-tarjeta.component.html',
  styles: [
  ]
})
export class UsuarioTarjetaComponent implements OnInit {
  @Input() usuario: any = {};//Recibir Datos de un Componente
  @Input() index: number | undefined;
  @Output() usuarioSeleccionado: EventEmitter<number>;//Enviar Datos a Otro Componente
  public url_files: String;

  constructor(private router: Router) { 
    this.url_files = GLOBAL.url_files;
    this.usuarioSeleccionado = new EventEmitter();
  }

  ngOnInit(): void {
  }
  
  verUsuario() {
    // console.log(  this.index );
    this.router.navigate( ['/usuario', this.usuario.username] );
    // this.usuarioSeleccionado.emit( this.index );
  }

} 

```

## Vista Componente Tarjeta de Usuario (usuario-tarjeta)

usuario-tarjeta.component.html:
```HTML
<div class="col">
    <div class="card">
        <img class="card-img-top" [src]="url_files+usuario.urlFoto" [alt]="usuario.nombreCompleto">
        <div class="card-body">
            <h5 class="card-title">{{ usuario.nombreCompleto }}</h5>
            <p class="card-text"> {{ usuario.profesion}} </p>
            <p class="card-text"><small class="text-muted">{{usuario.descripcionServicio }}</small></p>
            <button (click)="verUsuario()" type="button" class="btn btn-outline-primary btn-block">
            Ver más...
            </button>
            <!-- <a [routerLink]="['/heroe',i]" class="btn btn-outline-primary">Ver más link...</a> -->
        </div>
    </div>
</div>
```

## Controlador Componente Usuario

```JavaScript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router'
import { UsuarioService } from '../../../services/usuario.service';
import { GLOBAL } from '../../../services/global';

@Component({
  selector: 'app-usuario',
  templateUrl: './usuario.component.html',
  styles: [
  ]
})
export class UsuarioComponent implements OnInit {

  public usuario:any = {};
  public url_files: String
  constructor(private activatedRoute: ActivatedRoute,
    private _usuarioService: UsuarioService) {
      this.url_files = GLOBAL.url_files;
    }

  ngOnInit(): void {
    this.cargarUsuario();
  }

  cargarUsuario(): void{
    this.activatedRoute.params.subscribe(params => {
      let id = params['id']
      console.log(params)
      if(id){
        this._usuarioService.getUsuario(id).subscribe( (usuario) => this.usuario = usuario)
      }
    })
  }

}

```

## Vista Componente Usuario

```HTML
<h1 class="animated fadeIn">{{ usuario.nombreCompleto | uppercase }}</h1>
<hr>
<div class="row animated fadeIn fast">
    <div class="col-md-4">
        <img width="300px" height="350px" [src]="url_files+usuario.urlFoto" class="img-fluid" [alt]="usuario.nombreCompleto">
        <br><br>
        <br><br>
        <a [routerLink]="['/home']" class="btn btn-outline-danger btn-block">Regresar</a>
    </div>
    <div class="col-md-8">
        <h3>{{ usuario.profesion }}</h3>
        <hr>
        <p>
            {{ usuario.descripcionServicio }}
        </p>
    </div>
</div>
```
## Controlador Componente Home

```JavaScript
import { Component, OnInit } from '@angular/core';
import { UsuarioService } from '../../services/usuario.service';
import { Usuario } from '../../models/usuario';
import { Router } from '@angular/router';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html'
})
export class HomeComponent implements OnInit {
  usuarios:Usuario[] = [];
  constructor(private _usuarioService:UsuarioService,
    private router:Router) { }

  ngOnInit(): void {
    this._usuarioService.listasUsuarios().subscribe(data => (this.usuarios=data));
  }
  verUsuario( idx:number ){
    this.router.navigate( ['../usuario',idx] );  
  }

}

```
## Vista Componente Home

```HTML
<h1>Usuarios <small>Servicios vários a tu alcance</small></h1>
<hr>
<div class="row row-cols-1 row-cols-md-3 g-4">
    <app-usuario-tarjeta (usuarioSeleccionado)="verUsuario( $event )" [usuario]="foo" [index]="i" *ngFor="let foo of usuarios; let i = index"></app-usuario-tarjeta>
</div>
```
## Controlador del Componente Login

Vamos a iniciar codificando el controlador:

```JavaScript

```


## Vista Componente Login

Vamos a utilizar esta maqueta para el formulario login
```HTML
<h1 class="login-header">{{title}}</h1>
<div class="container">
    <div class="login">
        <div class="login-triangle"></div>
>
        <h2 class="login-header" center>Login</h2>
         <form  #formLogin="ngForm" class="login-container" (ngSubmit)="onLogin()">
            <p>
                <input type="text" name="username" #username="ngModel" [(ngModel)]="usuario.username"  placeholder="Nombre Usuario" ngModel required>
                <span *ngIf="!username.valid && username.touched">El Nombre de Usuario Es Obligatorio</span>
            </p>
            <p>
                <input type="password" name="password" #password="ngModel" [(ngModel)]="usuario.password"  placeholder="Password " ngModel required>
                <span *ngIf="!password.valid && password.touched">La Contraseña Es Obligatoria</span>
            </p>
            <input type="submit" value="Ingresar">
          </form> 
    </div>
</div>
```




