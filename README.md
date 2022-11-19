# ClientesApp

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.6.0.

## Servicio UploadFiles

```JavaScript
import { Injectable } from '@angular/core';
import { HttpClient, HttpRequest, HttpEvent,HttpParams} from '@angular/common/http';
import { Observable } from 'rxjs';
import { GLOBAL } from './global';
@Injectable({
  providedIn: 'root'
})
export class UploadFilesService {
  public url: String
  constructor(private _http: HttpClient){
    this.url = GLOBAL.ulr;
   
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
import { Component, OnInit } from '@angular/core';
import {Router, ActivatedRoute} from '@angular/router'

import { Usuario } from '../../../models/usuario'
import { UsuarioService } from '../../../services/usuario.service'
import { UploadFilesService } from '../../../services/upload-files.service'
import { HttpEventType, HttpResponse } from '@angular/common/http';
import { Observable } from 'rxjs';
@Component({
  selector: 'app-form',
  templateUrl: './form.component.html',
  styles: [
  ],
  providers: [UsuarioService]
})
export class FormComponent implements OnInit {
  public usuario: Usuario =  new Usuario('','','','','','')
  public titulo:string = "Crear Usuario"
  public mensaje:String="";
  public files:any;
  message = '';
  fileName = "";
  clave_encriptada = "";
  fileInfos:Observable<any> | undefined;
  constructor(
    private usuarioService: UsuarioService,
    private uploadFilesService: UploadFilesService,
    private router: Router,
    private activatedRoute: ActivatedRoute) { }

  ngOnInit(): void {
    this.cargarUsuario()
  }
  selectFiles(event:any) {
    //this.progressInfo = [];
    event.target.files.length == 1 ? this.fileName = event.target.files[0].name : this.fileName = event.target.files.length + " archivos";
    this.files = event.target.files;
  }
  uploadFiles() {
    this.message = '';
    this.upload(0, this.files[0]);
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

  create(formRegister:any): void {
    console.log(this.usuario);
    this.usuarioService.register(this.usuario).subscribe({
      next: (data) => {
        if(data){
          console.log(data)
          this.mensaje ="El Registro se Ha Realizado Correctamente...";
          this.usuario = new Usuario('','','','','','')
          alert(this.mensaje);
          formRegister.reset()
          this.router.navigate(['../usuarios'])
        }else{
         this.mensaje ="Error al Registrar Usuario";
       
        }
        console.log(this.mensaje);
        
      },
      error: (error) =>{
        console.log(<any>error);
      },
      complete: () => console.info('complete') 
    });
  }

  update():void{
    this.usuarioService.update(this.usuario)
    .subscribe( usuario => {
      this.router.navigate(['../usuarios'])
      alert(`Usuario ${this.usuario.nombreCompleto} actualizado con éxito!`)
      //swal('Cliente Actualizado', `Cliente ${cliente.nombre} actualizado con éxito!`, 'success')
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
import { Usuario } from '../../models/usuario';
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
  constructor(
    private _usuarioService: UsuarioService ) { }

  ngOnInit(): void {
    //if(!this._usuarioService.esAutenticado()){
    //  this._router.navigate(['login']);
    //}else{
      this._usuarioService.listarUsuarios().subscribe(data => (this.usuarios=data));
    //}
  }
  eliminar(usuario: Usuario): void {

    swal.fire({
      title: 'Está seguro?',
      text:`¿Seguro que desea eliminar al cliente ${usuario.nombreCompleto} ?`,
      icon: 'warning',
      showCancelButton: true,
      confirmButtonColor: '#3085d6',
      cancelButtonColor: '#d33',
      confirmButtonText: 'Si, eliminar!',
      cancelButtonText: 'No, cancelar!',
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

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `-prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).
