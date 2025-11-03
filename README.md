# ATM-Maneger
import tkinter as tk
from tkinter import messagebox
import mysql.connector

# -------------------- DATABASE CONFIGURATION --------------------
DB_HOST = "localhost"
DB_USER = "root"
DB_PASSWORD = "your_mysql_password"  # <-- CHANGE THIS
DB_DATABASE = "record"  # Must exist in MySQL
# ----------------------------------------------------------------


class ATMApp:
    """
    A simple ATM Management System GUI built with Tkinter and connected to MySQL.
    """
    def __init__(self, master):
        self.master = master
        master.title("ATM Machine Management System")

        self.current_atm_number = None
        self.current_account_number = None
        self.frames = {}

        # Connect to Database
        self.db_connection = self._connect_db()

        # Main container frame
        self.container = tk.Frame(master)
        self.container.pack(fill="both", expand=True, padx=20, pady=20)

        # Create frames
        self._create_login_frame()
        self._create_balance_frame()
        self._create_withdrawal_frame()
        self._create_transaction_frame()

        # Show login by default
        self.show_frame("Login")

    # -------------------- DATABASE CONNECTION --------------------
    def _connect_db(self):
        """Connect to MySQL database."""
        try:
            conn = mysql.connector.connect(
                host=DB_HOST,
                user=DB_USER,
                password=DB_PASSWORD,
                database=DB_DATABASE
            )
            messagebox.showinfo("DB Status", "Database connection successful!")
            return conn
        except mysql.connector.Error as err:
            messagebox.showerror("DB Error", f"Failed to connect to MySQL:\n{err}")
            return None

    # -------------------- FRAME NAVIGATION --------------------
    def show_frame(self, page_name):
        """Show the selected frame."""
        frame = self.frames[page_name]
        frame.tkraise()

    # -------------------- LOGIN FRAME --------------------
    def _create_login_frame(self):
        frame = tk.Frame(self.container, padx=40, pady=40, bg="#f0f0f0")
        frame.grid(row=0, column=0, sticky="nsew")
        self.frames["Login"] = frame

        tk.Label(frame, text="Welcome to ATM", font=('Arial', 24, 'bold'),
                 bg="#f0f0f0", fg="#0056b3").pack(pady=20)

        tk.Label(frame, text="ATM Card Number:", font=('Arial', 12), bg="#f0f0f0").pack(pady=5)
        self.atm_number_entry = tk.Entry(frame, width=30, font=('Arial', 12))
        self.atm_number_entry.pack(pady=5)

        tk.Label(frame, text="4-Digit PIN:", font=('Arial', 12), bg="#f0f0f0").pack(pady=5)
        self.pin_entry = tk.Entry(frame, width=30, show="*", font=('Arial', 12))
        self.pin_entry.pack(pady=5)

        tk.Button(frame, text="ENTER", command=self._login,
                  font=('Arial', 14, 'bold'), bg="#4CAF50", fg="white",
                  activebackground="#45a049", width=15, height=2).pack(pady=20)

        # Menu frame (post-login)
        menu_frame = tk.Frame(frame, bg="#e0e0e0", padx=10, pady=10)
        menu_frame.pack(pady=10)

        self.btn_balance = tk.Button(menu_frame, text="1. Balance Inquiry",
                                     command=lambda: self.show_frame("Balance"),
                                     font=('Arial', 12), width=20, state=tk.DISABLED)
        self.btn_withdraw = tk.Button(menu_frame, text="2. Withdrawal",
                                      command=lambda: self.show_frame("Withdrawal"),
                                      font=('Arial', 12), width=20, state=tk.DISABLED)
        self.btn_transaction = tk.Button(menu_frame, text="3. Transaction (Transfer)",
                                         command=lambda: self.show_frame("Transaction"),
                                         font=('Arial', 12), width=20, state=tk.DISABLED)
        self.btn_logout = tk.Button(menu_frame, text="4. Logout", command=self._logout,
                                    font=('Arial', 12), width=20, state=tk.DISABLED)

        self.btn_balance.grid(row=0, column=0, padx=5, pady=5)
        self.btn_withdraw.grid(row=1, column=0, padx=5, pady=5)
        self.btn_transaction.grid(row=2, column=0, padx=5, pady=5)
        self.btn_logout.grid(row=3, column=0, padx=5, pady=5)

    # -------------------- BALANCE FRAME --------------------
    def _create_balance_frame(self):
        frame = tk.Frame(self.container, padx=40, pady=40, bg="#e0f7fa")
        frame.grid(row=0, column=0, sticky="nsew")
        self.frames["Balance"] = frame

        tk.Label(frame, text="Account Balance Inquiry", font=('Arial', 20, 'bold'),
                 bg="#e0f7fa", fg="#0056b3").pack(pady=20)

        self.balance_label = tk.Label(frame, text="---", font=('Arial', 16), bg="#e0f7fa")
        self.balance_label.pack(pady=10)

        tk.Button(frame, text="Check Balance", command=self._check_balance,
                  font=('Arial', 14), bg="#00bcd4", fg="white", width=15).pack(pady=10)
        tk.Button(frame, text="Back to Menu", command=lambda: self.show_frame("Login"),
                  font=('Arial', 12), bg="#ff9800", fg="white", width=15).pack(pady=10)

    # -------------------- WITHDRAWAL FRAME --------------------
    def _create_withdrawal_frame(self):
        frame = tk.Frame(self.container, padx=40, pady=40, bg="#fff3e0")
        frame.grid(row=0, column=0, sticky="nsew")
        self.frames["Withdrawal"] = frame

        tk.Label(frame, text="Cash Withdrawal", font=('Arial', 20, 'bold'),
                 bg="#fff3e0", fg="#0056b3").pack(pady=20)

        tk.Label(frame, text="Enter Amount to Withdraw:", font=('Arial', 12), bg="#fff3e0").pack(pady=5)
        self.withdraw_amount_entry = tk.Entry(frame, width=30, font=('Arial', 12))
        self.withdraw_amount_entry.pack(pady=5)

        tk.Button(frame, text="Withdraw Cash", command=self._withdraw_cash,
                  font=('Arial', 14), bg="#f44336", fg="white", width=15).pack(pady=10)
        tk.Button(frame, text="Back to Menu", command=lambda: self.show_frame("Login"),
                  font=('Arial', 12), bg="#ff9800", fg="white", width=15).pack(pady=10)

    # -------------------- TRANSACTION FRAME --------------------
    def _create_transaction_frame(self):
        frame = tk.Frame(self.container, padx=40, pady=40, bg="#e8f5e9")
        frame.grid(row=0, column=0, sticky="nsew")
        self.frames["Transaction"] = frame

        tk.Label(frame, text="Fund Transfer", font=('Arial', 20, 'bold'),
                 bg="#e8f5e9", fg="#0056b3").pack(pady=20)

        tk.Label(frame, text="Recipient Account Number:", font=('Arial', 12), bg="#e8f5e9").pack(pady=5)
        self.recipient_acc_entry = tk.Entry(frame, width=30, font=('Arial', 12))
        self.recipient_acc_entry.pack(pady=5)

        tk.Label(frame, text="Amount to Transfer:", font=('Arial', 12), bg="#e8f5e9").pack(pady=5)
        self.transfer_amount_entry = tk.Entry(frame, width=30, font=('Arial', 12))
        self.transfer_amount_entry.pack(pady=5)

        tk.Button(frame, text="Transfer Funds", command=self._transfer_funds,
                  font=('Arial', 14), bg="#4CAF50", fg="white", width=15).pack(pady=10)
        tk.Button(frame, text="Back to Menu", command=lambda: self.show_frame("Login"),
                  font=('Arial', 12), bg="#ff9800", fg="white", width=15).pack(pady=10)

    # -------------------- LOGIC METHODS --------------------
    def _update_menu_buttons(self, state=tk.DISABLED):
        self.btn_balance.config(state=state)
        self.btn_withdraw.config(state=state)
        self.btn_transaction.config(state=state)
        self.btn_logout.config(state=state)

    def _login(self):
        if not self.db_connection:
            return

        atm_number = self.atm_number_entry.get()
        pin = self.pin_entry.get()

        if not atm_number or not pin:
            messagebox.showwarning("Input Error", "Please enter both ATM number and PIN.")
            return

        cursor = self.db_connection.cursor()
        query = "SELECT password, account_number FROM ATM WHERE ATM_Number = %s"

        try:
            cursor.execute(query, (atm_number,))
            result = cursor.fetchone()

            if result and result[0] == pin:
                self.current_atm_number = atm_number
                self.current_account_number = result[1]
                messagebox.showinfo("Success", "Login successful!")
                self._update_menu_buttons(state=tk.NORMAL)
                self.atm_number_entry.delete(0, tk.END)
                self.pin_entry.delete(0, tk.END)
            else:
                messagebox.showerror("Login Failed", "Invalid ATM Number or PIN.")
                self._update_menu_buttons(state=tk.DISABLED)
        except mysql.connector.Error as err:
            messagebox.showerror("Database Error", f"Login failed: {err}")
        finally:
            cursor.close()

    def _logout(self):
        self.current_atm_number = None
        self.current_account_number = None
        self._update_menu_buttons(state=tk.DISABLED)
        messagebox.showinfo("Logout", "You have been logged out.")
        self.show_frame("Login")

    def _get_balance(self):
        if not self.db_connection or not self.current_account_number:
            return None
        cursor = self.db_connection.cursor()
        query = "SELECT balance FROM ATM WHERE account_number = %s"
        try:
            cursor.execute(query, (self.current_account_number,))
            balance = cursor.fetchone()
            return balance[0] if balance else None
        except mysql.connector.Error as err:
            messagebox.showerror("Database Error", f"Balance retrieval failed: {err}")
            return None
        finally:
            cursor.close()

    def _check_balance(self):
        balance = self._get_balance()
        if balance is not None:
            self.balance_label.config(text=f"Current Balance: ${balance:,.2f}")
        else:
            self.balance_label.config(text="Could not fetch balance.")

    def _withdraw_cash(self):
        if not self.db_connection or not self.current_account_number:
            messagebox.showerror("Error", "Not logged in or DB not connected.")
            return

        try:
            amount = float(self.withdraw_amount_entry.get())
        except ValueError:
            messagebox.showerror("Input Error", "Please enter a valid amount.")
            return

        if amount <= 0:
            messagebox.showerror("Input Error", "Amount must be positive.")
            return

        cursor = self.db_connection.cursor()
        try:
            current_balance = self._get_balance()
            if amount > current_balance:
                messagebox.showerror("Failure", "Insufficient funds.")
                return

            new_balance = current_balance - amount
            update_query = "UPDATE ATM SET balance = %s WHERE account_number = %s"
            cursor.execute(update_query, (new_balance, self.current_account_number))
            self.db_connection.commit()

            messagebox.showinfo("Success", f"Withdrawal successful.\nNew Balance: ${new_balance:,.2f}")
            self.withdraw_amount_entry.delete(0, tk.END)
        except mysql.connector.Error as err:
            self.db_connection.rollback()
            messagebox.showerror("Database Error", f"Withdrawal failed: {err}")
        finally:
            cursor.close()

    def _transfer_funds(self):
        if not self.db_connection or not self.current_account_number:
            messagebox.showerror("Error", "Not logged in or DB not connected.")
            return

        recipient_acc = self.recipient_acc_entry.get()
        try:
            amount = float(self.transfer_amount_entry.get())
        except ValueError:
            messagebox.showerror("Input Error", "Please enter a valid amount.")
            return

        if amount <= 0:
            messagebox.showerror("Input Error", "Amount must be positive.")
            return
        if not recipient_acc or recipient_acc == str(self.current_account_number):
            messagebox.showerror("Input Error", "Please enter a valid recipient account.")
            return

        cursor = self.db_connection.cursor()
        try:
            # Check sender balance
            sender_balance = self._get_balance()
            if amount > sender_balance:
                messagebox.showerror("Failure", "Insufficient funds.")
                return

            # Check recipient existence
            cursor.execute("SELECT balance FROM ATM WHERE account_number = %s", (recipient_acc,))
            recipient_data = cursor.fetchone()
            if not recipient_data:
                messagebox.showerror("Failure", "Recipient account not found.")
                return

            # Perform transaction
            sender_new_balance = sender_balance - amount
            recipient_new_balance = recipient_data[0] + amount

            cursor.execute("UPDATE ATM SET balance = %s WHERE account_number = %s",
                           (sender_new_balance, self.current_account_number))
            cursor.execute("UPDATE ATM SET balance = %s WHERE account_number = %s",
                           (recipient_new_balance, recipient_acc))
            self.db_connection.commit()

            messagebox.showinfo("Success",
                                f"Transferred ${amount:,.2f} to {recipient_acc}.\nYour new balance: ${sender_new_balance:,.2f}")
            self.recipient_acc_entry.delete(0, tk.END)
            self.transfer_amount_entry.delete(0, tk.END)
        except mysql.connector.Error as err:
            self.db_connection.rollback()
            messagebox.showerror("Database Error", f"Transfer failed: {err}")
        finally:
            cursor.close()


# -------------------- MAIN APP START --------------------
if __name__ == '__main__':
    # Run these in MySQL once:
    # CREATE DATABASE record;
    # USE record;
    # CREATE TABLE ATM (
    #     account_number INT PRIMARY KEY,
    #     ATM_Number VARCHAR(20) UNIQUE,
    #     password VARCHAR(4),
    #     balance DECIMAL(10,2)
    # );
    # INSERT INTO ATM VALUES
    # (1234567, '0001234', '1234', 20000.00),
    # (7654321, '0005678', '5678', 30000.00);

    root = tk.Tk()
    app = ATMApp(root)
    root.geometry("500x600")
    root.mainloop()
