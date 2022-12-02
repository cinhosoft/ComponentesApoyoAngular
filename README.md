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
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute, Params} from '@angular/router';
import { Usuario } from 'src/app/models/usuario';
import { UsuarioService } from '../../../services/usuario.service';
import swal from 'sweetalert2';
@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent implements OnInit {
  public title: String;
  public usuario:Usuario; 
  constructor(
    private _route: ActivatedRoute,
    private _router: Router,
    private _usuarioService: UsuarioService) { 
      this.title = "Iniciar Sesión";
      this.usuario = new Usuario('','','','','','')
    }

  ngOnInit(): void {
    if(this._usuarioService.esAutenticado()){
      this._router.navigate(['home']);
    }else{
      this._router.navigate(['login']);
    }
  }

  onLogin():void{
    if(this.usuario.username == '' || this.usuario.password == ''){
      console.log("Error No digito datos");
      swal.fire('Error!', 'Error No digito datos completos' , 'error');
    }else{
      this._usuarioService.login(this.usuario.username, this.usuario.password).subscribe({
        next: (response)=>{
          this._usuarioService.guardarToken(response.token);
          this._usuarioService.guardarUsuario(response.idUsuario,response.nombreUsuario);
          this._router.navigate(['home'])
          swal.fire('Login!', 'Hola: ' + response.nombreUsuario  + ', ha iniciado correctamente' , 'success');
        },error: (error)=> {
          if(error.status == 401){
            swal.fire('Error Login!', 'Datos Digitados Son Incorrectos' , 'error');
          }
        }
      });
    }
 }
}
```

## Vista CSS Login

```CSS
html,
body {
    height: 100%;
}

body {
    display: flex;
    align-items: center;
    padding-top: 40px;
    padding-bottom: 40px;
    background-color: #f5f5f5;
}

.form-signin {
    max-width: 330px;
    padding: 15px;
}

.form-signin .form-floating:focus-within {
    z-index: 2;
}

.form-signin input[type="email"] {
    margin-bottom: -1px;
    border-bottom-right-radius: 0;
    border-bottom-left-radius: 0;
}

.form-signin input[type="password"] {
    margin-bottom: 10px;
    border-top-left-radius: 0;
    border-top-right-radius: 0;
}
```
## Vista HTML Login

Vamos a utilizar esta maqueta para el formulario login
```HTML
<main class="form-signin w-100 m-auto">
    <form #formLogin="ngForm" class="login-container" (ngSubmit)="onLogin()">
        <br><br><br>
        <h1 class="h3 mb-3 fw-normal">{{title}}</h1>
        <div class="form-floating">
            <input type="text" name="username" #username="ngModel" [(ngModel)]="usuario.username" class="form-control" id="floatingInput" placeholder="Username" ngModel required>
            <label for="floatingInput">Username</label>
            <span *ngIf="!username.valid && username.touched">El Nombre de Usuario Es Obligatorio</span>
        </div>
        <div class="form-floating">
            <input type="password" name="password" #password="ngModel" [(ngModel)]="usuario.password" class="form-control" id="floatingPassword" placeholder="Password" ngModel required>
            <label for="floatingPassword">Password</label>
            <span *ngIf="!password.valid && password.touched">La Contraseña Es Obligatoria</span>
        </div>
        <button class="w-100 btn btn-lg btn-primary" type="submit">Ingresar</button>
    </form>
</main>
```
## NavBar Componente

```JavaScript
import { Component, OnInit } from '@angular/core';
import { UsuarioService} from '../../../services/usuario.service';
import { Router } from '@angular/router';
import Swal from 'sweetalert2';
@Component({
  selector: 'app-navbar',
  templateUrl: './navbar.component.html'
})
export class NavbarComponent implements OnInit {

  title = 'Te Sirvo APP 17 - MISIONTIC';
  public logaedo: boolean=false;

  constructor(public _usuarioService:UsuarioService, private _router:Router) { }

  ngOnInit(): void {
  }
 
  logout():void {
    this._usuarioService.logout();
    Swal.fire("Logout", "Ha Cerrado Sesion con Exito","success");
    this._router.navigate(['login'])
  }
}
```


## NavBar con Autenticacion

Uso del metodo esAutenticado() y se agregan los botones de Login y Logout:

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
                    <a class="nav-link" *ngIf="_usuarioService.esAutenticado()" routerLink="usuarios" routerLinkActive="active">Administrador de Usuarios</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" routerLink="about" routerLinkActive="active">About</a>
                </li>
                <li *ngIf="!_usuarioService.esAutenticado()"><a class="nav-link" [routerLink]="['../usuarios/form']" routerLinkActive="active">Registrar</a> </li>

                <li class="nav-item" style="float:right" *ngIf="!_usuarioService.esAutenticado()">
                    <a style="float:right" class="nav-link" routerLink="login" routerLinkActive="active">Login</a>
                </li>

                <li class="nav-item" style="float:right" *ngIf="_usuarioService.esAutenticado()">
                    <a style="float:right" class="nav-link" (click)="logout()" routerLinkActive="active">Logout</a>
                </li>

                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                      Dropdown
                    </a>
                    <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
                        <li><a class="dropdown-item" href="#">Action</a></li>
                        <li><a class="dropdown-item" href="#">Another action</a></li>
                        <li>
                            <hr class="dropdown-divider">
                        </li>
                        <li><a class="dropdown-item" href="#">Something else here</a></li>
                    </ul>
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

## Service Completo
```JavaScript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders, HttpResponse } from  '@angular/common/http';
import { Observable } from 'rxjs';
import { GLOBAL } from './global';
import { Usuario } from '../models/usuario';

@Injectable({
  providedIn: 'root'
})
export class UsuarioService {
  private _token: string|null=null;
  private _username: string|null=null;
  private _idUsuario: string|null=null;
  public url: String
  private httpHeaders = new HttpHeaders({'Content-Type':'application/json'});
  constructor(private _http: HttpClient) {
    this.url = GLOBAL.url;
   }
   public get idUsuario():string|null{
    if(this._idUsuario !=null){
      return this._idUsuario;
    }else if((this._idUsuario==null && sessionStorage.getItem('id')!=null)){
      this._idUsuario = sessionStorage.getItem('id');
      return this._idUsuario;
    }
    return null;
   }
   public get username():string|null{
    if(this._username !=null){
      return this._username;
    }else if((this._username==null && sessionStorage.getItem('username')!=null)){
      this._username = sessionStorage.getItem('username');
      return this._username;
    }
    return null;
   }
   public get token():string|null{
    if(this._token !=null){
      return this._token;
    }else if((this._token==null && sessionStorage.getItem('token')!=null)){
      this._token = sessionStorage.getItem('token');
      return this._token;
    }
    return null;
   }

  login(username:String, password:String):Observable<any>{
    return this._http.post<any>(this.url+'authenticate',{username:username,password:password})
  }

  logout(){
    this._token=null;
    sessionStorage.clear();
  }

  guardarToken(token:string){
    this._token = token;
    sessionStorage.setItem('token',token);
  }

  guardarUsuario(id:string,nombreUsuario:string){
    this._idUsuario = id;
    this._username = nombreUsuario;
    sessionStorage.setItem('username',id);
    sessionStorage.setItem('nombreCompleto',nombreUsuario);
  }
  obtenerDatosToken(token:string):string|null{
    if(token!=null){
     return JSON.parse(atob(token.split(".")[1]));
    }
    return null;
  }
  esAutenticado():boolean{
    if(this.token != null){
      return true;
    }else{
      return false;
    }
  }
    
  //create
  register(user:Usuario):Observable<Usuario>{
    return this._http.post<Usuario>(this.url+'register',user,{headers:this.httpHeaders})
   }
  //read all
  listasUsuarios():Observable<any>{
    return this._http.get<any>(this.url+'user', {headers:this.httpHeaders});//GET->http://localhost:9011/user
   }
  //read one
  getUsuario(id:String):Observable<Usuario>{
    return this._http.get<Usuario>(`${this.url}user/${id}`, {headers:this.httpHeaders})
    //return this._http.get<any>(this.url+'user/'+id)
  }
  //update
  update(usuario: Usuario):Observable<Usuario>{
    return this._http.put<Usuario>(`${this.url}user/${usuario.username}`, usuario, {headers: this.httpHeaders})
  }
  //delete
  delete(id:String):Observable<Usuario> {
    return this._http.delete<Usuario>(this.url+'user/'+`${id}`, {headers:  this.httpHeaders});
   }
}

```




