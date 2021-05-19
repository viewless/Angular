# Angular Course: Cryptocurrencies

## Prerequisites

Install NodeJS & Angular CLI

[NodeJS](https://nodejs.org/en/)

Install the executable and after that check if it's correctly installed running in the console

```
node --version
v14.17.0

npm --version
6.14.13
```
Once you have NodeJS installed, proceed with the global installation of the Angular CLI (Command Line Interface)

```
npm install -g @angular/cli
```
Check the Angular CLI version
```
ng --version
Angular CLI: 12.0.0
```
---

## 1. Create a new Angular app using the CLI (Command Line Interface)

```
ng new lecture-exercise
```

1. Select Angular routing, type: `Y`
2. Select stylsheet format: `SCSS`

---
## 2. Enter in the newly created project in the console folder and install the dependencies runnings

```
cd lecture-exercise

npm install
```

---
## 3. Open the project with your favourite editor (`Visual Studio Code`, `Webstorm` etc ...)

All the important files are available in the `src/` folder

---
## 4. Run the new project (enter in the directory in the console and type)

```
npm start
```
You should be able to open a browser on url [localhost:4200](http://localhost:4200) and see the welcome page.

### Congrats! You shoul have now a running Angular app.
---


## 5. Create our first component `app-header`

We will store our components in the folder `src/app/components/`

The default folder where all the items (modules, components, services, directives, etc...) are generated is `src/app/`

In the console run (in the project folder):

```
ng generate component components/header
```
or the shortest version
```
ng g c components/header
```

---
## 6. Remove all the content in the `app.component.html` and put the code:

```html
<app-header></app-header>
```

Now in your browsert you should see the message `header works!`
Which means you successfully loaded the new header component.


# NOTE: All the CSS/SCSS should be copied because their implementation is not the goal of the cousre.

So for now you can copy or overwrite:

- `src/styles.scss`
- `src/app/header.component.scss`

from the folder `stylesheets`. They should have the all the same name!

---
## 7. Let's create a variable in the `app.component.ts` and pass it to the header:

- It should already exists named `title`. so give it a value of `Cryptocurrencies` and pass it to the header.

```ts
title = 'Cryptocurrencies';
```

- To pass it to the **header** first we need an `@Input` var in the `header.component.ts` to recieve the input.
- So in the `HeaderComponent` class create:

```ts
@Input() appTitle: string | undefined;
 ```

 It's also needed to include it in the imports from `@angular/core`. All decorators need to be imported.
 ```ts
 import { Component, Input, OnInit } from '@angular/core';
 ```

 Now in the **html** of the header we can get the passed input like:

 ```html
<h3 class="main-title">
  <span>{{appTitle}}</span>
</h3>
``` 

To pass it from the `app.component.html` we need to pass it to the `app-header`:
```html
<app-header [appTitle]="title"></app-header>
```

---
## 8. Now we should generate a new component, call it `Home` and give it a route. So:
```
ng g c components/home
```

Once it's created to give it a route we should import it in the `src/app/app-routing.module.ts`.
```ts
import { HomeComponent } from './components/home/home.component';
```

In the `routes` array we should add some routes:

```ts
const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: '**', component: HomeComponent }
];
```

1. The **first** object tells to redirect to `home` if no route is specified.
2. The **second** one means to load the `HomeComponent` when on `/home` route.
3. The **third** one is the wildcard that will redirect to `/home`, for any undefined url.

Finally to load the home component we need to include the `router-outlet` into the `app.component.html` like this.
```html
<app-header [appTitle]="title"></app-header>
<router-outlet></router-outlet>
```

---
## 9. Lets create the `home page` content

1. Copy the styles in the `src/app/components/home.component.scss` file.
2. Add the markup to the `src/app/components/home.component.html` file.
```html
<div class="content">

  <div class="portfolio-header">
    <h2 class="title">Portfolio</h2>
    <!-- <button (click)="loadData()">reload</button> -->
  </div>

</div>
```

---
## 10. Let's create a `service` which will load the data from an API endpoint
```
ng generate service services/cryptocurrencies
```
of the short version

```
ng g s services/cryptocurrencies
```

We should have a newly created service in the `src/app/services/cryptocurrencies.service.ts`

In the constructor of the **service** we need to inject the `HttpClient` as we're going to fetch some data with it. 

And of course don't foget to import it from `@angular/common/http`. So:
```ts
import { HttpClient } from '@angular/common/http';
```
```ts
constructor(private http: HttpClient) {}
```
### We should include the `HttpClientModule` in the `app.modiule.ts`

### In order to be able to use the service in the application we will need it in the main module, so provide it in the `providers` array in the `src/app/app.module.ts`. It should be imported as usual in the module first.

```ts
import { HttpClientModule } from '@angular/common/http';
import { CryptocurrenciesService } from './services/cryptocurrencies.service';
```

```ts
@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    HomeComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    AppRoutingModule
  ],
  providers: [CryptocurrenciesService],
  bootstrap: [AppComponent]
})
```


Next we need a method to fetch the data:
```ts
getCoinData(coinId: string): any {
  return this.http.get(`https://api.coingecko.com/api/v3/coins/${coinId}`);
}
```
Let's add some coins in a list in the `service class`:
```ts
coinsList = ['ethereum','bitcoin','ripple','cardano','litecoin','polkadot','dogecoin','shiba-inu'];
```

Next, we need to copy the `coin-data.ts` from the `models` folder and put it into `src/app/models/coin-data.ts`. It's a TypeScript interface.

Now back to the service, lets create a variable with the `loadedData` of type `CoinData` from the interface and import it. Let's put it below the `coinsList`.
```ts
import { CoinData } from '../models/coin-data';
```

```ts
loadedData: CoinData[] = [];
```

### What we're going to do next is to load the data, map the results and we'll show it in a table. Se let's declare two methods `loadTable()` and `mapResult()`:

```ts
loadTableData(): void {
  const requests: any[] = [];
  this.loadedData = [];

  this.coinsList.forEach((coinName) => {
    requests.push(this.getCoinData(coinName));
  });

  // forkJoin is part of RxJs so needs to be imported
  forkJoin(requests).subscribe(results => {
    results.map(result => {
      this.mapResult(result);
    });
  });
}

mapResult(coin: any): void {
  this.loadedData.push({
    id: coin.id,
    symbol: coin.symbol,
    name: coin.name,
    change_last_24h: coin.market_data.price_change_percentage_24h,
    value: coin.market_data.current_price.usd,
    last_updated_at: coin.market_data.last_updated,
    image: coin.image.thumb
  });
}
```

### What we did here is to put all the requests in an array and with `forkJoin` we combine all the requestts in a single subscribe, so we can have all the data at the same time.


---
## 11. Fetch the data and show it in a table into the `HomeComponent`.

In the `src/app/components/home.component.ts` create function which calls the service and loads the data.

1. First import the service and inject it into the component `constructor()`.

```ts
import { CryptocurrenciesService } from 'src/app/services/cryptocurrencies.service';
```
```ts
constructor(private cryptoService: CryptocurrenciesService) {}
```

2. Declare variable where to store the data and show it to the markup (Import the `CoinData` interface):
```ts
import { CoinData } from '../../models/coin-data';
```
In the `HomeComponent` class:
```ts
dataSource: CoinData[] = [];
```

3. Then create the `loadData()` method.

```ts
loadData(): void {
  this.cryptoService.loadTableData();
  this.dataSource = this.cryptoService.loadedData;
}
```
4. Finally call it when the component is initialized:
```ts
ngOnInit(): void {
  this.loadData();
}
```


In the `home.component.html` put the table below the **Portfolio header**.

```html
<table class="cryptos-table">
  <tr>
    <th>ID</th>
    <th></th>
    <th>Name</th>
    <th class="align-right">Price</th>
    <th class="align-right">24h</th>
    <!-- <th class="align-right">Updated at</th> -->
  </tr>
  <tr *ngFor="let coin of dataSource">
    <!-- Router Links variations -->

    <td>
      <!-- routerLink example -->
      <a href routerLink="/crypto/{{coin.id}}">{{ coin.symbol | uppercase }}</a>
    </td>

    <td>
      <img [src]="coin.image" alt="Coin logo">
    </td>

    <td>
      <!-- Another routerLink example -->
      <a [routerLink]="['/crypto',coin.id]">{{ coin.name }}</a>
    </td>

    <td class="align-right">{{coin.value | currency }}</td>
    <!-- <td class="align-right">{{ coin.change_last_24h | changed }}</td> -->
    <td class="align-right"><small>{{ coin.last_updated_at | date: 'HH:mm:ss dd.MM.yyyy' }}</small></td>
  </tr>
</table>
```
### Now you should see a table with some results fetched from the API endpoint.

### As you may observe there are some added `router links` which are broken, so let's fix them and add a new **component**

---
## 12. Create a `CryptoDetailsComponent` first.

```
ng g c components/crypto-details
```

### Now let's create the routes in `app-routing.module.ts` which redirect to this component. First inport the `CryptoDetailsComponent` then add it to the routes array.

```ts
import { CryptoDetailsComponent } from "./components/crypto-details/crypto-details.component";
```

```ts
const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'crypto/:id', component: CryptoDetailsComponent },
  { path: '**', component: HomeComponent }
];
```
### What we did is to pass to the route a parameter `id` which we can retrieve to load the data in the `CryptoDetailsComponent`.
### To make so in the component we need to import `ActivatedRoute` component.
### So in `src/app/components/crypto-details.component.ts` we need to import:

```ts
import { ActivatedRoute } from '@angular/router';
import { CryptocurrenciesService } from '../../services/cryptocurrencies.service';
```
then inject them in the `constructor()` of the copmponent and get the `id` from the route:

1. Declare the retrived `cryptoId` variable
```ts
cryptoId: string;
```
2. Inject and get store the `id` in `cryptoId`.

```ts
constructor(private activatedRoute: ActivatedRoute, private cryptoService: CryptocurrenciesService) {
  this.cryptoId = this.activatedRoute.snapshot.params.id;
}
```
3. Finally, display the id in the markup. So in `crypto-details.component.html` add:

```html
<h2>{{cryptoId.toUpperCase()}}</h2>
```
4. You may now add some styles. (`crypto-details.component.scss`).

5. Now that we've loaded the `id` of the route let's make use of it and load again the data from the service. So in the `ngOnInit()` we can load it.

  - Declare a new variable in the class to store the data;

  ```ts
  coinData: any;
  ```
  - Load the data from the service:
  ```ts
  ngOnInit(): void {
    this.cryptoService.getCoinData(this.cryptoId).subscribe((data: any) => {
      this.coinData = data;
    })
  };
  ```
  6. Display the image of the token and the price in the html:

  ```html
  <div *ngIf="coinData">
    <h2>{{coinData.market_data.current_price.usd | currency }}</h2>
    <img [src]="coinData.image.large" alt="Coin logo">
  </div>
  ```

---
## 13. Create a custom Pipe for the crypto change in the last 24h.

### To create a custom pipe run from the terminal in the project directory:
```
ng generate pipe pipes/changed
```
or the short version

```
ng g p pipes/changed
```

### Once created lets modify it. So what we want this pipe to do is to display the percentage of the changes from the last 24h.
### Let's do this in `src/app/pipes/changed.pipe.ts`. Simply replace the `transform()` method with`:

```ts
transform(value: number): any {
  const arrow: string = value <= 0 ? '↓' : '↑';
  return `${value.toFixed(2)}% ${arrow}`;
}
```
### Now lets activate it in the table. Uncomment the following lines in `home.component.html`:
```html
<!-- <th class="align-right">Updated at</th> -->
<!-- <td class="align-right">{{ coin.change_last_24h | changed }}</td> -->
```

---
## 14. Add a Directive to change the color of the daily balance in percentage of the tokens.

### For that purpose we will start with adding a directive to our project (from the terminal int the project root):

```
ng generate directive directives/changeColor
```
or for short

```
ng g d directives/changeColor
```
### A directive is similar to a component as its applied to it.

1. We need to import the `@Input` decorator, the `ElementRef` (element reference) and `OnInit` method;
```ts
import { Directive, ElementRef, Input, OnInit } from '@angular/core';
``` 
so the directive shoul look like this:

```ts
@Directive({
  selector: '[appChangeColor]'
})
export class ChangeColorDirective implements OnInit {

  @Input() changeValue: number = 0;

  constructor(private elementRef: ElementRef) {}

  ngOnInit(): void {
    this.elementRef.nativeElement.style.color = this.changeValue < 0 ? 'crimson' : 'forestgreen';
  }
}
```

2. To apply the directive, in the `home.component.html` update to the following line;

```html
<td class="align-right" appChangeColor [changeValue]="coin.change_last_24h">{{ coin.change_last_24h | changed }}</td>
```

---
## 15. We can add a link to `/home` in the header to be able to go back to the `HomeComponent` when on a `crypto deitails` page.

### To do so, in the `header.component.html` file we can add:

```html
<a routerLink="/home">Home</a>
```

### just below the app title, so finally we obtain:

```html
<h3 class="main-title">
  <span>{{appTitle}}</span>
  <a routerLink="/home">Home</a>
</h3>
```

---
# Thank You!