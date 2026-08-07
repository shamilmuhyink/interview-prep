# Module 4: Frontend Architecture (Angular)

> **Scope:** Angular Lifecycle, State Management, Rendering Strategies, Component Architecture, Backend Integration, RxJS
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 Explain the Angular component lifecycle hooks. Which ones do you use most frequently and why?

**Angular components go through a defined lifecycle from creation to destruction, with hooks like `ngOnInit`, `ngOnChanges`, `ngOnDestroy`, and `ngAfterViewInit` allowing you to tap into specific moments — `ngOnInit` for initialization logic and `ngOnDestroy` for cleanup are the two you'll use in virtually every component.**

**Lifecycle Order:**

| # | Hook | When Called | Common Use |
|---|------|-----------|------------|
| 1 | `ngOnChanges` | Before `ngOnInit` and whenever `@Input` changes | React to input changes |
| 2 | `ngOnInit` | Once, after first `ngOnChanges` | Fetch data, initialize subscriptions |
| 3 | `ngDoCheck` | Every change detection cycle | Custom change detection |
| 4 | `ngAfterContentInit` | After content projection (`<ng-content>`) | Access projected content |
| 5 | `ngAfterContentChecked` | After every projected content check | — |
| 6 | `ngAfterViewInit` | After view and child views initialized | Access `@ViewChild`, DOM manipulation |
| 7 | `ngAfterViewChecked` | After every view check | — |
| 8 | `ngOnDestroy` | Before component is destroyed | Unsubscribe, cleanup timers |

```typescript
@Component({
  selector: 'app-order-detail',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <div *ngIf="order">
      <h2>{{ order.id }}</h2>
      <canvas #chart></canvas>
    </div>
  `
})
export class OrderDetailComponent implements OnInit, OnChanges, AfterViewInit, OnDestroy {
  @Input({ required: true }) orderId!: string;
  @ViewChild('chart') chartRef!: ElementRef<HTMLCanvasElement>;
  
  order: Order | null = null;
  private destroy$ = new Subject<void>();

  constructor(private orderService: OrderService) {}

  // ① Called when @Input changes (including initial set)
  ngOnChanges(changes: SimpleChanges): void {
    if (changes['orderId'] && !changes['orderId'].firstChange) {
      this.loadOrder(); // Re-fetch when orderId input changes
    }
  }

  // ② Called once after first ngOnChanges — initialization logic here
  ngOnInit(): void {
    this.loadOrder();
  }

  // ③ Called after view is initialized — DOM is available
  ngAfterViewInit(): void {
    this.initChart(this.chartRef.nativeElement);
  }

  // ④ Cleanup — prevent memory leaks
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  private loadOrder(): void {
    this.orderService.getOrder(this.orderId).pipe(
      takeUntil(this.destroy$) // Auto-unsubscribe on destroy
    ).subscribe(order => this.order = order);
  }
}
```

**⚠️ Pitfalls:**
- **Don't put logic in the constructor** — `@Input` values are not yet set. Use `ngOnInit`.
- **`@ViewChild` is not available in `ngOnInit`** — it's only set by `ngAfterViewInit`.
- **Not unsubscribing in `ngOnDestroy`** causes memory leaks. Use `takeUntil(destroy$)`, the `async` pipe, or `DestroyRef`.
- `ngDoCheck` and `ngAfterViewChecked` are called on EVERY change detection cycle — expensive logic here kills performance.

---

### Q2. 🔴 🏢 What is Angular Change Detection? Explain Default vs. OnPush strategies.

**Change Detection is Angular's mechanism for synchronizing the component's data model with the DOM — the Default strategy checks the entire component tree on every event, while OnPush only checks a component when its `@Input` references change, an event fires within it, or an Observable emits via the `async` pipe — dramatically reducing re-renders.**

**Default Strategy:**
- Checks EVERY component in the tree from root to leaves.
- Triggered by: any browser event, timer, HTTP response, promise resolution.
- Simple but expensive for large component trees.

**OnPush Strategy:**
- Checks the component ONLY when:
  1. An `@Input` reference changes (not deep mutations).
  2. An event originates from the component or its children.
  3. An Observable bound with `async` pipe emits.
  4. `ChangeDetectorRef.markForCheck()` is called manually.

```typescript
@Component({
  selector: 'app-product-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush, // ← Enable OnPush
  template: `
    <div *ngFor="let product of products; trackBy: trackById">
      <app-product-card [product]="product" />
    </div>
    <p>Total: {{ total$ | async }}</p>  <!-- async pipe triggers OnPush check -->
  `
})
export class ProductListComponent {
  @Input() products: Product[] = [];
  total$ = this.store.select(selectProductTotal);

  trackById(index: number, product: Product): string {
    return product.id;
  }
}

// ⚠️ WRONG: Mutating array doesn't trigger OnPush
this.products.push(newProduct);  // Same reference — OnPush won't detect

// ✅ CORRECT: Create new array reference
this.products = [...this.products, newProduct];  // New reference — triggers check
```

**Angular Signals (17+) — The Future:**
```typescript
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `<p>Count: {{ count() }}</p>
             <button (click)="increment()">+1</button>`
})
export class CounterComponent {
  count = signal(0);
  doubled = computed(() => this.count() * 2); // Derived reactive value

  increment() {
    this.count.update(v => v + 1); // Fine-grained reactivity — no zone.js needed
  }
}
```

**⚠️ Pitfalls:**
- With OnPush, calling `this.data.property = newValue` inside a subscription won't update the view — call `this.cdr.markForCheck()` or use the `async` pipe.
- `trackBy` in `*ngFor` is essential for OnPush performance — without it, Angular recreates DOM elements unnecessarily.
- Angular Signals (v17+) are moving toward Zone-less change detection — future Angular versions may deprecate zone.js.

---

### Q3. 🔴 🏢 How do you manage state in Angular applications? Compare service-based, NgRx, and signal-based approaches.

**Angular state management ranges from simple injectable services with BehaviorSubjects (small apps) to NgRx store with actions/reducers/effects (complex, enterprise) to Angular Signals (17+, lightweight reactive state) — choose based on app complexity and team familiarity.**

| Approach | Complexity | Best For | Predictability |
|----------|-----------|---------|---------------|
| Service + BehaviorSubject | Low | Small-medium apps | Medium |
| NgRx (Redux pattern) | High | Large enterprise apps | Very high (time-travel debugging) |
| Signals + Signal Store | Medium | New apps (Angular 17+) | High |

```typescript
// 1. Service-based state (BehaviorSubject)
@Injectable({ providedIn: 'root' })
export class CartService {
  private cartItems$ = new BehaviorSubject<CartItem[]>([]);
  readonly items$ = this.cartItems$.asObservable();
  readonly total$ = this.items$.pipe(
    map(items => items.reduce((sum, i) => sum + i.price * i.quantity, 0))
  );

  addItem(item: CartItem): void {
    const current = this.cartItems$.getValue();
    const existing = current.find(i => i.productId === item.productId);
    if (existing) {
      this.cartItems$.next(
        current.map(i => i.productId === item.productId 
          ? { ...i, quantity: i.quantity + 1 } : i)
      );
    } else {
      this.cartItems$.next([...current, { ...item, quantity: 1 }]);
    }
  }
}

// 2. NgRx (Redux pattern — actions + reducers + effects)
// actions
export const loadProducts = createAction('[Products] Load');
export const loadProductsSuccess = createAction('[Products] Load Success', props<{ products: Product[] }>());
export const loadProductsFailure = createAction('[Products] Load Failure', props<{ error: string }>());

// reducer
export const productsReducer = createReducer(
  initialState,
  on(loadProducts, state => ({ ...state, loading: true })),
  on(loadProductsSuccess, (state, { products }) => ({ ...state, products, loading: false })),
  on(loadProductsFailure, (state, { error }) => ({ ...state, error, loading: false }))
);

// effect — side effects (API calls)
@Injectable()
export class ProductEffects {
  loadProducts$ = createEffect(() => this.actions$.pipe(
    ofType(loadProducts),
    exhaustMap(() => this.productService.getAll().pipe(
      map(products => loadProductsSuccess({ products })),
      catchError(error => of(loadProductsFailure({ error: error.message })))
    ))
  ));
}

// 3. Signal Store (Angular 17+ with @ngrx/signals)
export const ProductStore = signalStore(
  withState<ProductState>({ products: [], loading: false, error: null }),
  withComputed((store) => ({
    productCount: computed(() => store.products().length),
    activeProducts: computed(() => store.products().filter(p => p.active)),
  })),
  withMethods((store, productService = inject(ProductService)) => ({
    async loadProducts() {
      patchState(store, { loading: true });
      try {
        const products = await firstValueFrom(productService.getAll());
        patchState(store, { products, loading: false });
      } catch (error) {
        patchState(store, { error: (error as Error).message, loading: false });
      }
    }
  }))
);
```

**⚠️ Pitfalls:**
- NgRx adds significant boilerplate — don't use it for simple CRUD apps.
- BehaviorSubject state can become inconsistent if multiple components update simultaneously — no action log for debugging.
- Signals are still evolving — `@ngrx/signals` (Signal Store) is the recommended pattern for new projects.

---

### Q4. 🔴 🌐 How do you integrate Angular with a Spring Boot backend? Explain HTTP client patterns and error handling.

**Angular uses the `HttpClient` module to communicate with Spring Boot REST APIs — production patterns include centralized error handling with interceptors, typed response models, retry logic, and authentication token injection.**

```typescript
// HTTP service with proper error handling
@Injectable({ providedIn: 'root' })
export class OrderService {
  private readonly apiUrl = '/api/v1/orders';

  constructor(private http: HttpClient) {}

  getOrders(params: OrderSearchParams): Observable<Page<OrderSummary>> {
    return this.http.get<Page<OrderSummary>>(this.apiUrl, {
      params: {
        page: params.page.toString(),
        size: params.size.toString(),
        ...(params.status && { status: params.status })
      }
    });
  }

  createOrder(request: CreateOrderRequest): Observable<Order> {
    return this.http.post<Order>(this.apiUrl, request);
  }

  updateStatus(orderId: string, status: OrderStatus): Observable<Order> {
    return this.http.patch<Order>(`${this.apiUrl}/${orderId}/status`, { status });
  }
}

// Auth interceptor — inject JWT token into every request
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token && !req.url.includes('/auth/login')) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        authService.logout();
        inject(Router).navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
};

// Error interceptor — centralized error handling
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const notification = inject(NotificationService);

  return next(req).pipe(
    retry({
      count: 2,
      delay: (error, retryCount) => {
        if (error.status === 0 || error.status >= 500) {
          return timer(1000 * retryCount); // Exponential backoff for server errors
        }
        return throwError(() => error); // Don't retry client errors (4xx)
      }
    }),
    catchError((error: HttpErrorResponse) => {
      const message = error.error?.detail || error.message || 'An error occurred';
      notification.showError(message);
      return throwError(() => error);
    })
  );
};

// Register interceptors in app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    )
  ]
};
```

**CORS Configuration (Spring Boot side):**
```java
@Configuration
public class CorsConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:4200")
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true);
            }
        };
    }
}
```

**⚠️ Pitfalls:**
- **Proxy in development** — use `proxy.conf.json` to avoid CORS issues during development: `{ "/api/*": { "target": "http://localhost:8080" } }`.
- **Always unsubscribe from HTTP observables** in components — `HttpClient` completes automatically, but switchMap/mergeMap chains may not.
- **Don't expose stack traces in API errors** — Spring Boot's `ProblemDetail` format is clean and production-safe.

---

### Q5. 🔴 🌐 What is lazy loading in Angular? How does it affect performance?

**Lazy loading defers the loading of Angular modules/components until they're navigated to — reducing the initial bundle size and Time-to-Interactive (TTI), which is the single most impactful performance optimization for large Angular applications.**

```typescript
// Route-level lazy loading (standalone components — Angular 15+)
export const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'orders',
    loadComponent: () => import('./orders/order-list.component')
      .then(m => m.OrderListComponent),
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES),
    canActivate: [adminGuard],
  },
  {
    path: 'reports',
    loadComponent: () => import('./reports/reports.component')
      .then(m => m.ReportsComponent),
    data: { preload: true }  // Custom preload strategy flag
  }
];

// Custom preload strategy — preload routes marked with data.preload
@Injectable({ providedIn: 'root' })
export class SelectivePreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}

// Register in app config
provideRouter(routes, withPreloading(SelectivePreloadStrategy))

// Lazy loading a heavy component (not route-based)
@Component({
  template: `
    @defer (on viewport) {
      <app-heavy-chart [data]="chartData" />
    } @placeholder {
      <div class="skeleton-chart"></div>
    } @loading (minimum 300ms) {
      <app-spinner />
    }
  `
})
export class DashboardComponent { }
```

**Performance Impact:**
| Metric | Before Lazy Loading | After Lazy Loading |
|--------|--------------------|--------------------|
| Initial bundle | 2.5 MB | 800 KB |
| TTI | 4.2s | 1.8s |
| Routes loaded | All upfront | On demand |

**⚠️ Pitfalls:**
- **Shared modules** loaded in multiple lazy chunks get duplicated — extract them into a shared chunk.
- **Preloading strategy matters** — `PreloadAllModules` defeats the purpose for large apps. Use selective preloading.
- `@defer` (Angular 17+) is the modern way to lazy-load components without routing — replaces heavy `*ngIf` patterns.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🌐 What is RxJS and how do you use Observables effectively in Angular?

**RxJS is the reactive extensions library that Angular uses for asynchronous data streams — Observables are the foundation for HTTP calls, event handling, and state management, with operators like `switchMap`, `debounceTime`, and `combineLatest` enabling complex async composition.**

```typescript
// Search with debounce — THE classic RxJS pattern
@Component({
  template: `<input [formControl]="searchControl">
             <div *ngFor="let result of results$ | async">{{ result.name }}</div>`
})
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results$!: Observable<Product[]>;

  ngOnInit() {
    this.results$ = this.searchControl.valueChanges.pipe(
      debounceTime(300),                          // Wait 300ms after last keystroke
      distinctUntilChanged(),                     // Don't re-search same term
      filter(term => term!.length >= 2),          // Min 2 chars
      switchMap(term => this.productService.search(term!).pipe(
        catchError(() => of([]))                  // Graceful error handling
      ))
    );
    // switchMap cancels the previous HTTP request if a new term arrives
  }
}
```

**Key Operators to Know:**

| Operator | Use Case |
|---------|----------|
| `switchMap` | Cancel previous, use latest (search, navigation) |
| `mergeMap` | Execute all concurrently (batch operations) |
| `concatMap` | Execute sequentially, maintain order (queued operations) |
| `exhaustMap` | Ignore new until current completes (form submit — prevent double-click) |
| `combineLatest` | Combine multiple streams, emit when any changes |
| `forkJoin` | Wait for all to complete (parallel API calls) |

**⚠️ Pitfalls:**
- **Memory leaks** — unsubscribed observables keep running. Use `takeUntil(destroy$)`, `async` pipe, or `takeUntilDestroyed()` (Angular 16+).
- **`mergeMap` for HTTP** can fire dozens of requests simultaneously — use `switchMap` for searches, `exhaustMap` for form submits.
- **Cold vs. Hot** — `HttpClient` observables are cold (execute per subscription). `BehaviorSubject` is hot (shared state).

---

### Q7. 🌐 How do Angular Reactive Forms differ from Template-Driven Forms?

**Reactive Forms define the form model programmatically in TypeScript, giving full control over validation, dynamic fields, and testing — Template-Driven Forms define the model in the template with directives, suitable only for simple, static forms.**

| Feature | Reactive Forms | Template-Driven Forms |
|---------|---------------|----------------------|
| Model definition | Component class | Template (ngModel) |
| Validation | Programmatic (Validators) | Directive-based |
| Dynamic fields | Easy (FormArray) | Difficult |
| Testability | Easy (no DOM needed) | Requires TestBed |
| Immutability | Immutable data flow | Mutable (two-way binding) |

```typescript
@Component({
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  template: `
    <form [formGroup]="orderForm" (ngSubmit)="onSubmit()">
      <input formControlName="customerName" />
      <div *ngIf="orderForm.get('customerName')?.errors?.['required'] 
                   && orderForm.get('customerName')?.touched">
        Name is required
      </div>

      <div formArrayName="items">
        <div *ngFor="let item of items.controls; let i = index" [formGroupName]="i">
          <input formControlName="productName" />
          <input formControlName="quantity" type="number" />
          <button type="button" (click)="removeItem(i)">Remove</button>
        </div>
      </div>
      <button type="button" (click)="addItem()">Add Item</button>

      <button type="submit" [disabled]="orderForm.invalid">Submit</button>
    </form>
  `
})
export class OrderFormComponent {
  orderForm = new FormGroup({
    customerName: new FormControl('', [Validators.required, Validators.minLength(2)]),
    items: new FormArray([this.createItemGroup()])
  });

  get items(): FormArray { return this.orderForm.get('items') as FormArray; }

  createItemGroup(): FormGroup {
    return new FormGroup({
      productName: new FormControl('', Validators.required),
      quantity: new FormControl(1, [Validators.required, Validators.min(1)])
    });
  }

  addItem(): void { this.items.push(this.createItemGroup()); }
  removeItem(i: number): void { this.items.removeAt(i); }

  // Custom cross-field validator
  dateRangeValidator(group: FormGroup): ValidationErrors | null {
    const start = group.get('startDate')?.value;
    const end = group.get('endDate')?.value;
    return start && end && start > end ? { dateRange: true } : null;
  }
}
```

**⚠️ Pitfall:** Don't mix Reactive and Template-Driven in the same form — `ngModel` with `formControlName` triggers a warning and causes confusion.

---

### Q8. 🏢 What are Angular standalone components and why are they important?

**Standalone components (Angular 14+, default in 17+) are self-contained components that declare their own dependencies via `imports` instead of requiring an `NgModule` — simplifying the developer experience, reducing boilerplate, and enabling better tree-shaking.**

```typescript
// Standalone component — no NgModule needed
@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatCardModule,
    UserAvatarComponent,     // Import other standalone components directly
    DateFormatPipe            // Import standalone pipes directly
  ],
  template: `
    <mat-card>
      <app-user-avatar [user]="user" />
      <p>Joined: {{ user.createdAt | dateFormat }}</p>
    </mat-card>
  `
})
export class UserProfileComponent {
  @Input({ required: true }) user!: User;
}

// Bootstrapping without AppModule (Angular 17+ default)
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimationsAsync(),
  ]
});
```

**⚠️ Pitfall:** When migrating from NgModules, remember that each standalone component must import its own dependencies — there's no shared `declarations` array to inherit from.

---

### Q9. 🌐 How do you handle routing and route guards in Angular?

**Angular Router maps URL paths to components with support for lazy loading, nested routes, and route guards — guards (functional since Angular 15+) control access via `canActivate`, `canDeactivate`, `canMatch`, and `resolve` to enforce auth, unsaved changes prompts, and data preloading.**

```typescript
// Functional guard (Angular 15+ — replaces class-based guards)
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  
  return router.createUrlTree(['/login'], { 
    queryParams: { returnUrl: state.url } 
  });
};

// Role-based guard
export const roleGuard: CanActivateFn = (route) => {
  const authService = inject(AuthService);
  const requiredRoles = route.data['roles'] as string[];
  return requiredRoles.some(role => authService.hasRole(role));
};

// Route resolver — preload data before navigation
export const orderResolver: ResolveFn<Order> = (route) => {
  const orderService = inject(OrderService);
  const orderId = route.paramMap.get('id')!;
  return orderService.getOrder(orderId).pipe(
    catchError(() => {
      inject(Router).navigate(['/orders']);
      return EMPTY;
    })
  );
};

// Routes configuration
export const routes: Routes = [
  {
    path: 'orders',
    canActivate: [authGuard],
    children: [
      { path: '', loadComponent: () => import('./order-list.component').then(m => m.OrderListComponent) },
      {
        path: ':id',
        loadComponent: () => import('./order-detail.component').then(m => m.OrderDetailComponent),
        resolve: { order: orderResolver }
      }
    ]
  },
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard],
    data: { roles: ['ADMIN'] },
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  }
];
```

**⚠️ Pitfall:** `canDeactivate` guards are essential for forms with unsaved changes — without them, users lose data on accidental navigation.

---

### Q10. 🏢 What are Angular Signals and how do they change reactive programming?

**Signals (Angular 16+) are a fine-grained reactivity primitive that tracks dependencies automatically — when a signal's value changes, only the specific UI bindings that read that signal are updated, eliminating zone.js overhead and enabling predictable, synchronous reactivity.**

```typescript
@Component({
  selector: 'app-shopping-cart',
  standalone: true,
  template: `
    <div *ngFor="let item of cartItems()">
      {{ item.name }} — {{ item.quantity }}
      <button (click)="updateQuantity(item.id, item.quantity + 1)">+</button>
    </div>
    <p>Total: {{ totalPrice() | currency }}</p>
    <p>Tax: {{ tax() | currency }}</p>
    <p>Items: {{ itemCount() }}</p>
  `
})
export class ShoppingCartComponent {
  // Writable signals
  cartItems = signal<CartItem[]>([]);
  taxRate = signal(0.08);

  // Computed signals — derived, cached, auto-updated
  itemCount = computed(() => this.cartItems().length);
  totalPrice = computed(() => 
    this.cartItems().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  tax = computed(() => this.totalPrice() * this.taxRate());

  // Effects — side effects when signals change
  constructor() {
    effect(() => {
      // Runs whenever cartItems() or totalPrice() changes
      console.log(`Cart updated: ${this.itemCount()} items, total: ${this.totalPrice()}`);
      localStorage.setItem('cart', JSON.stringify(this.cartItems()));
    });
  }

  updateQuantity(itemId: string, newQty: number): void {
    this.cartItems.update(items =>
      items.map(i => i.id === itemId ? { ...i, quantity: newQty } : i)
    );
  }
}
```

**Signals vs. Observables:**

| Feature | Signals | RxJS Observables |
|---------|---------|-----------------|
| Sync/Async | Synchronous | Asynchronous |
| Current value | Always available via `signal()` | Need BehaviorSubject |
| Operators | `computed()`, `effect()` | 100+ operators |
| Glitch-free | Yes (batched updates) | No (diamond problem) |
| Use case | UI state, derived values | Event streams, HTTP, WebSocket |

**⚠️ Pitfall:** Signals and Observables complement each other — use `toSignal()` and `toObservable()` to bridge between them. Don't try to replace all RxJS with signals.

---

### Q11. 🏢 How do you optimize Angular application performance?

**Angular performance optimization focuses on three pillars: reducing bundle size (lazy loading, tree-shaking), minimizing change detection cycles (OnPush, trackBy, Signals), and efficient rendering (`@defer`, virtual scrolling, image optimization).**

```typescript
// 1. OnPush + trackBy — reduce change detection
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="list-viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackById">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
export class LargeListComponent {
  trackById = (index: number, item: Item) => item.id;
}

// 2. @defer for viewport-triggered lazy loading (Angular 17+)
@Component({
  template: `
    @defer (on viewport) {
      <app-heavy-analytics-dashboard />
    } @placeholder {
      <div class="h-96 skeleton"></div>
    }
  `
})
export class PageComponent {}

// 3. NgOptimizedImage for automatic image optimization
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `<img ngSrc="hero.jpg" width="800" height="400" priority />`
})
export class HeroComponent {}
```

**Key Optimizations Checklist:**
- ✅ Use `OnPush` change detection everywhere
- ✅ Lazy load routes and heavy components
- ✅ Use `trackBy` with `*ngFor`
- ✅ Virtual scrolling for long lists (`@angular/cdk/scrolling`)
- ✅ Avoid expensive computations in templates — use `computed()` or pipes
- ✅ Enable production build: `ng build --configuration=production`
- ✅ Use `budgets` in `angular.json` to catch bundle bloat

---

### Q12. 🏬 What is Content Projection and how does `ng-content` work?

**Content projection (`ng-content`) allows a parent component to inject template content into designated slots of a child component — enabling flexible, reusable container components like cards, modals, and layouts without tightly coupling their internal structure.**

```typescript
// Reusable card component with named slots
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[cardTitle]"></ng-content>
        <ng-content select="[cardActions]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>  <!-- Default slot -->
      </div>
      <div class="card-footer">
        <ng-content select="[cardFooter]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {}

// Usage — parent projects content into slots
<app-card>
  <h3 cardTitle>Order #12345</h3>
  <button cardActions (click)="edit()">Edit</button>
  
  <p>Order details go here...</p>  <!-- Goes into default slot -->
  
  <div cardFooter>
    <button (click)="save()">Save</button>
  </div>
</app-card>
```

**⚠️ Pitfall:** `ng-content` does NOT lazy-render projected content — even if wrapped in `*ngIf="false"`, the projected components are created. Use `ng-template` + `ngTemplateOutlet` for conditional projection.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🏬 How do you handle environment-specific configuration in Angular?

**Angular uses `environment.ts` files replaced at build time for environment-specific settings — combined with runtime configuration via `APP_INITIALIZER` for values that shouldn't be baked into the bundle (API URLs, feature flags).**

```typescript
// environment.ts (development)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api',
  features: { darkMode: true, betaFeatures: true }
};

// Runtime config loading (preferred for Docker/K8s deployments)
export function initializeApp(http: HttpClient) {
  return () => http.get<AppConfig>('/assets/config.json')
    .pipe(tap(config => AppConfigService.setConfig(config)))
    .toPromise();
}

// Register in app.config.ts
provideAppInitializer(() => {
  const http = inject(HttpClient);
  return firstValueFrom(http.get<AppConfig>('/assets/config.json')
    .pipe(tap(config => inject(AppConfigService).setConfig(config))));
})
```

**⚠️ Pitfall:** `environment.ts` values are baked into the bundle at build time — for Docker deployments where the same image runs in multiple environments, use runtime config loading.

---

### Q14. 🏬 What are Angular Pipes and when should you create custom ones?

**Pipes transform data in templates without modifying the underlying value — Angular provides built-in pipes (`date`, `currency`, `async`) and supports custom pure pipes (cached, only re-evaluate when input changes) and impure pipes (re-evaluate every CD cycle).**

```typescript
@Pipe({ name: 'timeAgo', standalone: true, pure: true })
export class TimeAgoPipe implements PipeTransform {
  transform(value: Date | string | number): string {
    const seconds = Math.floor((Date.now() - new Date(value).getTime()) / 1000);
    if (seconds < 60) return 'just now';
    if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
    if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
    return `${Math.floor(seconds / 86400)}d ago`;
  }
}

// Usage
<span>{{ order.createdAt | timeAgo }}</span>
```

**⚠️ Pitfall:** Impure pipes (`pure: false`) run on EVERY change detection cycle — they're as expensive as method calls in templates. Avoid them.

---

### Q15. 🌐 How do you write unit tests for Angular components?

**Angular testing uses Jasmine + Karma (or Jest) with `TestBed` for component testing — best practices include testing behavior (not implementation), using `spectator` for simpler setup, and isolating components with mock services.**

```typescript
describe('OrderListComponent', () => {
  let spectator: Spectator<OrderListComponent>;
  const createComponent = createComponentFactory({
    component: OrderListComponent,
    imports: [HttpClientTestingModule],
    providers: [
      { provide: OrderService, useValue: { 
        getOrders: () => of([{ id: '1', status: 'ACTIVE', total: 99.99 }]) 
      }}
    ]
  });

  beforeEach(() => spectator = createComponent());

  it('should display orders', () => {
    expect(spectator.queryAll('.order-row')).toHaveLength(1);
    expect(spectator.query('.order-total')).toHaveText('$99.99');
  });

  it('should call delete and refresh list', () => {
    const service = spectator.inject(OrderService);
    spyOn(service, 'deleteOrder').and.returnValue(of(void 0));
    spectator.click('.delete-button');
    expect(service.deleteOrder).toHaveBeenCalledWith('1');
  });
});
```

---

### Q16. 🏢 What is Server-Side Rendering (SSR) in Angular and when should you use it?

**Angular SSR (using Angular Universal / `@angular/ssr`) renders pages on the server and sends pre-rendered HTML to the client — improving First Contentful Paint (FCP) and SEO for content-heavy pages, while hydration preserves interactivity.**

```typescript
// Enable SSR in Angular 17+ project
// ng add @angular/ssr

// server.ts (auto-generated)
import { CommonEngine } from '@angular/ssr';
const commonEngine = new CommonEngine();

app.get('*', (req, res) => {
  commonEngine.render({
    bootstrap: AppServerModule,
    documentFilePath: indexHtml,
    url: req.originalUrl,
  }).then(html => res.send(html));
});
```

**When to Use:**
- ✅ Public-facing content pages (SEO critical)
- ✅ Marketing/landing pages (fast FCP)
- ❌ Dashboard/admin apps (no SEO needed, adds complexity)

**⚠️ Pitfall:** `document`, `window`, and `localStorage` are not available on the server — use `isPlatformBrowser()` guards or `afterNextRender()`.

---

### Q17. 🌐 How do you implement authentication flows in Angular (JWT + Spring Security)?

**Angular JWT authentication involves storing tokens in memory (not localStorage) or httpOnly cookies, attaching them via HTTP interceptors, and implementing auth guards to protect routes — paired with Spring Security's stateless JWT validation on the backend.**

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private currentUser = signal<User | null>(null);
  private tokenExpiry = signal<number>(0);
  readonly isAuthenticated = computed(() => 
    this.currentUser() !== null && Date.now() < this.tokenExpiry()
  );

  login(credentials: LoginRequest): Observable<void> {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.currentUser.set(response.user);
        this.tokenExpiry.set(Date.now() + response.expiresIn * 1000);
        // Store token in memory only — not localStorage (XSS vulnerable)
        this.tokenStore.set(response.accessToken);
        this.scheduleTokenRefresh(response.expiresIn);
      }),
      map(() => void 0)
    );
  }

  private scheduleTokenRefresh(expiresInSec: number): void {
    // Refresh 60 seconds before expiry
    timer((expiresInSec - 60) * 1000).pipe(
      switchMap(() => this.http.post<AuthResponse>('/api/auth/refresh', {}))
    ).subscribe(response => {
      this.tokenStore.set(response.accessToken);
      this.scheduleTokenRefresh(response.expiresIn);
    });
  }
}
```

**⚠️ Pitfall:** **Never store JWT in `localStorage`** — it's accessible to any XSS attack. Use httpOnly cookies (set by the server) or in-memory storage with refresh tokens.

---

### Q18. 🏬 What are Angular Directives and when do you create custom ones?

**Directives add behavior to existing DOM elements — structural directives (`*ngIf`, `*ngFor`) alter the DOM layout, attribute directives modify element appearance/behavior, and custom directives encapsulate reusable DOM interactions.**

```typescript
// Custom attribute directive — click-outside detection
@Directive({ selector: '[appClickOutside]', standalone: true })
export class ClickOutsideDirective {
  @Output() appClickOutside = new EventEmitter<void>();

  @HostListener('document:click', ['$event.target'])
  onDocumentClick(target: HTMLElement): void {
    if (!this.elementRef.nativeElement.contains(target)) {
      this.appClickOutside.emit();
    }
  }

  constructor(private elementRef: ElementRef) {}
}

// Usage
<div class="dropdown" (appClickOutside)="closeDropdown()">
  ...dropdown content...
</div>
```

---

### Q19. 🏬 How do you handle internationalization (i18n) in Angular?

**Angular supports i18n via built-in `@angular/localize` (compile-time, generates separate bundles per locale) or runtime libraries like `ngx-translate` (single bundle, dynamic locale switching) — choose compile-time for performance, runtime for flexibility.**

```typescript
// ngx-translate (runtime — more common in SPAs)
// app.config.ts
provideTranslateService({
  loader: { provide: TranslateLoader, useFactory: HttpLoaderFactory, deps: [HttpClient] },
  defaultLanguage: 'en'
})

// Component
<h1>{{ 'WELCOME_MESSAGE' | translate: { name: user.name } }}</h1>

// en.json
{ "WELCOME_MESSAGE": "Welcome, {{name}}!" }
// es.json
{ "WELCOME_MESSAGE": "¡Bienvenido, {{name}}!" }
```

---

### Q20. 🏬 What is the difference between `ViewChild`, `ContentChild`, and their plural variants?

**`@ViewChild` queries elements in the component's own template; `@ContentChild` queries elements projected via `<ng-content>` — the plural variants (`@ViewChildren`, `@ContentChildren`) return `QueryList` for multiple matches.**

```typescript
@Component({
  template: `
    <input #searchInput />                         <!-- ViewChild target -->
    <app-tab *ngFor="let tab of tabs" [data]="tab" />  <!-- ViewChildren target -->
    <ng-content></ng-content>                      <!-- ContentChild targets come from parent -->
  `
})
export class TabContainerComponent implements AfterViewInit, AfterContentInit {
  @ViewChild('searchInput') searchInput!: ElementRef;
  @ViewChildren(TabComponent) viewTabs!: QueryList<TabComponent>;
  @ContentChildren(TabComponent) contentTabs!: QueryList<TabComponent>;

  ngAfterViewInit() { this.searchInput.nativeElement.focus(); }  // ViewChild available
  ngAfterContentInit() { this.contentTabs.forEach(tab => tab.init()); }  // ContentChild available
}
```

**⚠️ Pitfall:** `@ViewChild` is available in `ngAfterViewInit`, NOT in `ngOnInit`. `@ContentChild` is available in `ngAfterContentInit`.
