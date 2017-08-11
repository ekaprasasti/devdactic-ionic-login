# Devdactic Ionic 2 Login

Catatan pembelajaran mengenai login app ionic 2 dari [devdactic.com](https://devdactic.com/login-ionic-2/) 

## Setting Ionic App

Buka terminal, ketikan perintah berikut satu persatu:

```bash
ionic start devdactic-simpleLogin blank
cd devdactic-simpleLogin
ionic g provider authService
ionic g page register
ionic g page login
```

Buka file `src/app/app.module.ts` dan ganti kodenya menjadi seperti berikut:

```javascript
import { AuthService } from './../providers/auth-service';
import { BrowserModule } from '@angular/platform-browser';
import { ErrorHandler, NgModule } from '@angular/core';
import { IonicApp, IonicErrorHandler, IonicModule } from 'ionic-angular';
import { SplashScreen } from '@ionic-native/splash-screen';
import { StatusBar } from '@ionic-native/status-bar';
 
import { MyApp } from './app.component';
 
@NgModule({
  declarations: [
    MyApp
  ],
  imports: [
    BrowserModule,
    IonicModule.forRoot(MyApp)
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp
  ],
  providers: [
    StatusBar,
    SplashScreen,
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    AuthService
  ]
})
export class AppModule {}
```

Buka file `src/app/app.component.ts` dan ganti homepage dengan `LoginPage`:

```javascript
import { Component } from '@angular/core';
import { Platform } from 'ionic-angular';
import { StatusBar } from 'ionic-native';
import { SplashScreen } from '@ionic-native/splash-screen';
 
@Component({
  templateUrl: 'app.html'
})
export class MyApp {
  rootPage:any = 'LoginPage';
 
  constructor(platform: Platform, statusBar: StatusBar, splashScreen: SplashScreen) {
    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      statusBar.styleDefault();
      splashScreen.hide();
    });
  }
}
```

## Membuat Authentication Service

Buka file `src/providers/auth-service.ts` dan edit kodenya menjadi seperti berikut:

```javascript
import { Injectable } from '@angular/core';
import {Observable} from 'rxjs/Observable';
import 'rxjs/add/operator/map';
 
export class User {
  name: string;
  email: string;
 
  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}
 
@Injectable()
export class AuthService {
  currentUser: User;
 
  public login(credentials) {
    if (credentials.email === null || credentials.password === null) {
      return Observable.throw("Please insert credentials");
    } else {
      return Observable.create(observer => {
        // At this point make a request to your backend to make a real check!
        let access = (credentials.password === "pass" && credentials.email === "email");
        this.currentUser = new User('Simon', 'saimon@devdactic.com');
        observer.next(access);
        observer.complete();
      });
    }
  }
 
  public register(credentials) {
    if (credentials.email === null || credentials.password === null) {
      return Observable.throw("Please insert credentials");
    } else {
      // At this point store the credentials to your backend!
      return Observable.create(observer => {
        observer.next(true);
        observer.complete();
      });
    }
  }
 
  public getUserInfo() : User {
    return this.currentUser;
  }
 
  public logout() {
    return Observable.create(observer => {
      this.currentUser = null;
      observer.next(true);
      observer.complete();
    });
  }
}
```

## Membuat Login

Akan berisi 2 input dan 2 button, input akan terkoneksi dengan objek `registerCredentials` dan akan di lewatkan ke `authService` ketika kita menekan login.

Setelah login kita akan di antarkan ke `HomePage`.

Buka file `src/pages/login/login.ts` dan edit kodenya menjadi seperti ini:

```javascript
import { Component } from '@angular/core';
import { NavController, AlertController, LoadingController, Loading, IonicPage } from 'ionic-angular';
import { AuthService } from '../../providers/auth-service';
 
@IonicPage()
@Component({
  selector: 'page-login',
  templateUrl: 'login.html',
})
export class LoginPage {
  loading: Loading;
  registerCredentials = { email: '', password: '' };
 
  constructor(private nav: NavController, private auth: AuthService, private alertCtrl: AlertController, private loadingCtrl: LoadingController) { }
 
  public createAccount() {
    this.nav.push('RegisterPage');
  }
 
  public login() {
    this.showLoading()
    this.auth.login(this.registerCredentials).subscribe(allowed => {
      if (allowed) {        
        this.nav.setRoot('HomePage');
      } else {
        this.showError("Access Denied");
      }
    },
      error => {
        this.showError(error);
      });
  }
 
  showLoading() {
    this.loading = this.loadingCtrl.create({
      content: 'Please wait...',
      dismissOnPageChange: true
    });
    this.loading.present();
  }
 
  showError(text) {
    this.loading.dismiss();
 
    let alert = this.alertCtrl.create({
      title: 'Fail',
      subTitle: text,
      buttons: ['OK']
    });
    alert.present(prompt);
  }
}
```

Sekarang kita akan membuat viewnya, buka file `src/pages/login/login.html` dan edit kodenya menjadi seperti ini:

```html
<ion-content class="login-content" padding>
  <ion-row class="logo-row">
    <ion-col></ion-col>
    <ion-col width-67>
      <img src="http://placehold.it/300x200"/>
    </ion-col>
    <ion-col></ion-col>
  </ion-row>
  <div class="login-box">
    <form (ngSubmit)="login()" #registerForm="ngForm">
      <ion-row>
        <ion-col>
          <ion-list inset>
            
            <ion-item>
              <ion-input type="text" placeholder="Email" name="email" [(ngModel)]="registerCredentials.email" required></ion-input>
            </ion-item>
            
            <ion-item>
              <ion-input type="password" placeholder="Password" name="password" [(ngModel)]="registerCredentials.password" required></ion-input>
            </ion-item>
            
          </ion-list>
        </ion-col>
      </ion-row>
      
      <ion-row>
        <ion-col class="signup-col">
          <button ion-button class="submit-btn" full type="submit" [disabled]="!registerForm.form.valid">Login</button>
          <button ion-button class="register-btn" block clear (click)="createAccount()">Create New Account</button>
        </ion-col>
      </ion-row>
      
    </form>
  </div>
</ion-content>
```

Kita akan menstylingnya, buka file `src/pages/login/login.scss` dan edit kodenya menjadi seperti berikut:

```css
page-login {
 
  .login-content {
    background: #56CA96;
 
    .logo-row {
      padding-top: 50px;
      padding-bottom: 20px;
    }
 
    .login-box {
      background: #399F8B;
      padding: 20px 20px 0px 20px;
      margin-top: 30px;
    }
 
    ion-row {
       align-items: center;
       text-align: center;
     }
 
     ion-item {
         border-radius: 30px !important;
         padding-left: 30px !important;
         font-size: 0.9em;
         margin-bottom: 10px;
         border: 1px solid #ffffff;
         border-bottom: 0px !important;
         box-shadow: none !important;
     }
 
     .signup-col {
       margin: 0px 16px 0px 16px;
       padding-bottom: 20px;
     }
 
     .item-inner {
       border-bottom-color: #ffffff !important;
       box-shadow: none !important;
     }
 
     .submit-btn {
       background: #51CFB1;
       border-radius: 30px !important;
       border: 1px solid #ffffff;
     }
 
     .register-btn {
       color: #ffffff;
       font-size: 0.8em;
     }
  }
 
}
```

Login screen kita akan terlihat seperti berikut saat ini:

![login screen](https://devdactic.com/wp-content/uploads/2016/10/ionic-2-login-template.png)



