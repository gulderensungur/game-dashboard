# Game Dashboard Application

First I create a project `ng new game-dashboard —style=scss` .

Then I add materials with `ng add @angular/material` and gauge(speedometer) with `npm install --save angular-gauge`

After some installation, I import modules in app.module.

```tsx
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";

import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
import { BrowserAnimationsModule } from "@angular/platform-browser/animations";
import { FormsModule } from "@angular/forms";
import { HttpClientModule } from "@angular/common/http";

import { GaugeModule } from "angular-gauge";
import { MatTabsModule } from "@angular/material/tabs";
import { MatIconModule } from "@angular/material/icon";
import { MatFormFieldModule } from "@angular/material/form-field";
import { MatSelectModule } from "@angular/material/select";

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    HttpClientModule,
    FormsModule,
    MatFormFieldModule,
    MatTabsModule,
    MatIconModule,
    MatSelectModule,
    GaugeModule.forRoot(),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

This is the latest version of app.modul.ts

And now, I create a search-bar component, add some codes in html part and I call it in app.component. Then I add styling with SCSS. This is my first use of SCSS.

```tsx
<div class="search-container">
  <form class="search-form" #form="ngForm" (ngSubmit)="onSubmit(form)">
    <span class="logo" routerLink="/">NgVGDB</span>
    <input
      class="search-input"
      type="text"
      name="search"
      ngModel
      placeholder="Search 500,000+ games"
    />
    <button class="search-button">Search</button>
  </form>
</div>
```

Here is my search-bar html file. I added the `#form="ngForm"` and `ngSubmit()` method. When I enter a text in search bar, text is submitted and navigated to defined path.

But first I need to define the `onSubmit()` function in search-bar.ts file.

```tsx
export class SearchBarComponent implements OnInit {
  constructor(private router: Router) {}

  ngOnInit(): void {}

  onSubmit(form: NgForm) {
    this.router.navigate(["search", form.value.search]);
  }
}
```

In onSubmit function take a form parameter and when the form submit, it is navigated due to the value of the search.

After the Searchbar component, I created Homepage Component. Then I added the path and element of the homepage component to router.module.ts.

```tsx
const routes: Routes = [
  {
    path: "",
    component: HomepageComponent,
  },
  {
    path: "search/:game-search",
    component: HomepageComponent,
  },
];
```

I call the `<router-outlet></router-outlet>` in app.component.html

The second route is after the searching process. Now, we set the Homepage component. We will add the filter part and game card part to the Homepage.

Firstly, I created a filter component and I added angular material in it.

```tsx
<div class="filters">
  <mat-form-field>
    <mat-label>Sort</mat-label>
    <mat-select
    panelClass="sort-select" [(ngModel)] = "sort">
      <mat-option value="name">Name</mat-option>
      <mat-option value="-released">Released</mat-option>
      <mat-option value="-added">Added</mat-option>
      <mat-option value="-created">Created</mat-option>
      <mat-option value="-updated">Updated</mat-option>
      <mat-option value="-rating">Rating</mat-option>
      <mat-option value="metacritic">Metacritic</mat-option>
    </mat-select>
  </mat-form-field>
</div>
```

I’m not sure that how to work these materials, so I will look into detail later on.

Then I called this component in the Homepage component like that: `<app-filter></app-filter>`

Now I will create Gamecard component and I will also call it in Homepage.

---

Now I implement the `getGameList()` function in http.servise.ts.

```tsx
getGameList(
    ordering:string,
    search?:string
    ):
    Observable<APIResponse<Game>>{
      let params = new HttpParams().set("ordering", ordering);

      if(search){
        params = new HttpParams().set("ordering", ordering).set("search", search);
      }
      return this.http.get<APIResponse<Game>>(`${env.BASE_URL}/games`,{
        params:params});
  }
```

On the above code, function takes a ordering and search parameter and it returns object that type APIResponse<Game>. I will define these types in Interface soon. We set the params according to wheter there is a search parameter or not. Also we created BASE_URL in environment file.

Now, lets create Interface.

```tsx
export interface Game {
  background_image: string;
  name: string;
  released: string;
  metacritic_url: string;
	.
	.

}
.
.

```

### **Angular Custom Interceptor:**

Interceptors are used to manage to requests.

> [Uygulama içinde yapılan istek ve cevaplarda araya girerek gerekli yönlendirme, log atma ya da gelip giden verilere değer ekleme gibi işlemleri yapabiliriz yani trafiği yönetebiliriz.](https://sametcelikbicak.com/angular-custom-interceptor)

**Here is our HttpErrorsInterceptor.**

```tsx
import { Injectable } from "@angular/core";
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
} from "@angular/common/http";
import { Observable, throwError } from "rxjs";
import { catchError } from "rxjs/operators";

@Injectable()
export class HttpErrorsInterceptor implements HttpInterceptor {
  constructor() {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((err) => {
        console.log(err);
        return throwError(() => new Error(err));
      })
    );
  }
}
```

### **The `next` object:**

The `next` object represents the next interceptor in the chain of interceptors. The final `next` in the chain is the `[HttpClient](https://angular.io/api/common/http/HttpClient)` backend handler that sends the request to the server and receives the server's response.

Most interceptors call `next.handle()`so that the request flows through to the next interceptor and, eventually, the backend handler.

**Note:**

In the source video, `observableThrowError()` has used for catching errors. But I learned that this function is deprecated so we use `throwError(() => new Error(err))` instead of `observableThrowError()`.

**Here is our HttpHeadersInterceptor**

```tsx
import { Injectable } from "@angular/core";
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
} from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable()
export class HttpHeadersInterceptor implements HttpInterceptor {
  constructor() {}

  intercept(
    request: HttpRequest<unknown>,
    next: HttpHandler
  ): Observable<HttpEvent<unknown>> {
    request = request.clone({
      setHeaders: {
        "X-RapidAPI-Key": "0bf0d9d33emsh6d0faac8a1b6423p1875ecjsn0c901ee38893",
        "X-RapidAPI-Host": "rawg-video-games-database.p.rapidapi.com",
      },
      setParams: {
        key: "5753b5c93846486d9fffa637773bf395",
      },
    });
    return next.handle(request);
  }
}
```

> The `clone()`method's hash argument lets you mutate specific properties of the request while copying the others.

We created interceptors. Now we need to insert and import our application. So we need to go to the app.module.ts and we import in providers.
