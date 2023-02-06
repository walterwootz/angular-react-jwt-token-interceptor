# Front-end JWT Token Interceptor for Angular & React
The purpose of JWT interceptor is to add a Bearer JWT token to all http calls throughout the application automatically, if a token exists in the session storage. These examples covers 2 use cases:
- Angular + [@angular/common/http](https://angular.io/api/common/http)
- React + [axios](https://www.npmjs.com/package/axios)

In the Angular use case you can find also a reponse handler that redirect to login page if token has expired (401 http status code).

## Angular
Just create a `token.interceptor.ts` as shown below.

```ts
import { Injectable } from '@angular/core';
import {
    HttpRequest,
    HttpHandler,
    HttpEvent,
    HttpInterceptor, HttpErrorResponse
} from '@angular/common/http';
import { Observable } from 'rxjs';
import {tap} from "rxjs/operators";
import {Router} from "@angular/router";

@Injectable()
export class TokenInterceptor implements HttpInterceptor {

    constructor(private router: Router) {}

    intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        // except for /login endpoint
        if (request.url.includes('/login')) {
            return next.handle(request);
        }
        // edit request
        request = request.clone({
            // bring token from sessionStorage and add as header
            setHeaders: {
                Authorization: `Bearer ${sessionStorage.getItem("token")}`
            }
        });
        return next.handle(request).pipe( tap(() => {},
            (err: any) => {
                // handle response
                if (err instanceof HttpErrorResponse) {
                    if (err.status !== 401) {
                        return false;
                    }
                    console.warn('Session expired. Please login again.', 'danger').then(null);
                    sessionStorage.removeItem('token');
                    this.router.navigate(['login']);
                }
        }));
    }

}

```


## React
Create a `token.interceptor.js` as shown below and call the function `jwtInterceptor()` in App.js or index.js.

```js
import axios from "axios";

export function jwtInterceptor() {
    axios.interceptors.request.use(async request => {
        // add auth header with jwt if token is set
        const token = sessionStorage.getItem("token");
        // except for /login endpoint
        if (!request.url.includes('/login') && token !== null && token !== undefined) {
            request.headers.common.Authorization = `Bearer ${token}`;
        }
        return request;
    });
}
```
