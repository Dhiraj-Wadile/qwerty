
import { Component } from '@angular/core';
import { Router } from '@angular/router';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as fromCustomer from '../../store/customer.reducer';
import * as CustomerActions from '../../store/customer.actions';
import { ApiService } from '../../core/api.service';

@Component({
  selector: 'app-login',
  template: `
    <div class="login-wrapper">
      <div class="login-box">
        <h2>Bank Login</h2>

        <div *ngIf="errorMessage" class="error-box">{{ errorMessage }}</div>

        <form (submit)="login($event)">
          <input [(ngModel)]="id" name="id" placeholder="Customer ID" required />
          <input [(ngModel)]="pin" name="pin" type="password" placeholder="PIN" required />
          <button type="submit">Login</button>
        </form>

        <div style="text-align: center; margin-top: 10px;">
          <button class="forgot-id-btn" (click)="toggleForgotId()">Forgot Customer ID?</button>
        </div>

        <!-- Forgot ID Modal -->
        <div *ngIf="isForgotIdVisible" class="forgot-id-modal">
          <div class="modal-content">
            <h3>Retrieve Customer ID</h3>
            <p>Please enter your registered email or phone number to retrieve your Customer ID.</p>
            <input [(ngModel)]="forgotIdInput" placeholder="Email / Phone" />

            <div class="modal-actions">
              <button (click)="forgotId()">Submit</button>
              <button class="close-btn" (click)="toggleForgotId()">Close</button>
            </div>

            <div *ngIf="customerId" class="customer-id-display">
              <p>Your Customer ID is: <strong>{{ customerId }}</strong></p>
            </div>
          </div>
        </div>

        <div *ngIf="isLoggedIn()" class="logout-wrapper">
          <button class="logout-btn" (click)="logout()">Logout</button>
        </div>
      </div>
    </div>
  `,
  styles: [`
    .login-wrapper {
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      background: linear-gradient(to right, #e0eafc, #cfdef3);
      font-family: 'Segoe UI', sans-serif;
    }

    .login-box {
      background-color: #ffffff;
      padding: 40px 30px;
      border-radius: 12px;
      box-shadow: 0px 15px 25px rgba(0, 0, 0, 0.1);
      width: 100%;
      max-width: 400px;
      transition: all 0.3s ease-in-out;
    }

    .login-box h2 {
      text-align: center;
      margin-bottom: 25px;
      color: #002244;
    }

    .login-box input {
      width: 100%;
      padding: 12px;
      margin-bottom: 18px;
      border: 1px solid #ccc;
      border-radius: 6px;
      font-size: 15px;
    }

    .login-box input:focus {
      outline: none;
      border: 1px solid #004080;
      box-shadow: 0 0 5px rgba(0, 64, 128, 0.3);
    }

    .login-box button {
      width: 100%;
      padding: 12px;
      background-color: #004080;
      color: white;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }

    .login-box button:hover {
      background-color: #002244;
    }

    .forgot-id-btn {
      background: none;
      border: none;
      color: #004080;
      text-decoration: none;
      font-size: 14px;
      cursor: pointer;
      padding: 0;
    }

    .forgot-id-btn:hover {
      color: #002244;
    }

    .error-box {
      background-color: #ffe5e5;
      color: #cc0000;
      padding: 10px 15px;
      margin-bottom: 20px;
      border-radius: 6px;
      text-align: center;
      font-weight: bold;
    }

    .logout-wrapper {
      margin-top: 20px;
      text-align: center;
    }

    .logout-btn {
      background-color: #aa0000;
      color: white;
      border: none;
      padding: 10px 20px;
      font-size: 14px;
      border-radius: 6px;
      cursor: pointer;
    }

    .logout-btn:hover {
      background-color: #880000;
    }

    /* Forgot ID Modal Styles */
    .forgot-id-modal {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.5);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }

    .modal-content {
      background-color: #ffffff;
      padding: 25px;
      border-radius: 10px;
      width: 90%;
      max-width: 400px;
      text-align: center;
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    }

    .modal-content h3 {
      font-size: 20px;
      margin-bottom: 15px;
      color: #004080;
    }

    .modal-content p {
      font-size: 14px;
      margin-bottom: 20px;
      color: #666;
    }

    .modal-content input {
      width: 100%;
      padding: 12px;
      margin-bottom: 20px;
      border: 1px solid #ccc;
      border-radius: 6px;
      font-size: 15px;
    }

    .modal-content input:focus {
      border: 1px solid #004080;
      box-shadow: 0 0 5px rgba(0, 64, 128, 0.3);
    }

    .modal-actions {
      display: flex;
      flex-direction: column;
      gap: 10px;
      align-items: center;
    }

    .modal-actions button {
      width: 100%;
      padding: 12px;
      font-size: 14px;
      font-weight: bold;
      border-radius: 6px;
      border: none;
      cursor: pointer;
    }

    .modal-actions button:first-child {
      background-color: #004080;
      color: white;
    }

    .modal-actions button:first-child:hover {
      background-color: #002244;
    }

    .close-btn {
      background-color: #888;
      color: white;
    }

    .close-btn:hover {
      background-color: #555;
    }

    .customer-id-display {
      margin-top: 20px;
      background-color: #e0f7fa;
      padding: 15px;
      border-radius: 6px;
      color: #00695c;
      font-weight: bold;
    }
  `]
})
export class LoginComponent {
  id!: number;
  pin!: string;
  errorMessage: string = '';
  forgotIdInput: string = '';
  customerId: string | null = null;
  isForgotIdVisible: boolean = false;

  // Store Observable for getting login state
  isLoggedIn$: Observable<boolean>;

  constructor(
    private apiService: ApiService,
    private router: Router,
    private store: Store<fromCustomer.CustomerState>
  ) {
    // Select login state from store
    this.isLoggedIn$ = this.store.select(fromCustomer.selectIsLoggedIn);
  }

  login(event: Event): void {
    event.preventDefault();
    this.errorMessage = '';

    this.apiService.login(this.id, this.pin).subscribe({
      next: (res) => {
        this.apiService.saveToken(res.token);
        localStorage.setItem('customer', JSON.stringify(res.customer));
        this.store.dispatch(CustomerActions.loginSuccess({ id: res.customer.id }));
        this.router.navigate(['/dashboard']);
      },
      error: (err) => {
        this.errorMessage = 'Invalid Customer ID or PIN. Please try again.';
        console.error(err);
      }
    });
  }

  forgotId(): void {
    if (this.forgotIdInput && this.forgotIdInput.trim()) {
      this.errorMessage = '';
      this.apiService.forgotId(this.forgotIdInput.trim()).subscribe({
        next: (res) => {
          if (res && res.customerId) {
            this.customerId = res.customerId;
          } else {
            this.errorMessage = '❌ No Customer ID found for the given details.';
          }
        },
        error: (err) => {
          this.errorMessage = '❌ Error while fetching Customer ID.';
          console.error(err);
        }
      });
    }
  }

  toggleForgotId(): void {
    this.isForgotIdVisible = !this.isForgotIdVisible;
    this.customerId = null;
  }

  logout(): void {
    localStorage.removeItem('customer');
    this.store.dispatch(CustomerActions.logout());
    this.router.navigate(['/login']);
  }

  isLoggedIn(): boolean {
    return !!localStorage.getItem('customer');
  }
}
