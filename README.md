# ClientesApp

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.6.0.

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
Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `-prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).
