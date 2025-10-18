# How to Extract Design Artifacts from the Implementation

This guide shows you exactly how to create your required design documents based on the implemented code.

---

## 1️⃣ Use Case Descriptions

### Template:
```
Use Case ID: UC-XX
Use Case Name: [Name from user story]
Actor: User Admin
Description: [From user story]
Preconditions: [What must be true before]
Postconditions: [What is true after]
Main Flow:
  1. [Step 1]
  2. [Step 2]
  ...
Alternative Flows:
  [If something goes wrong]
```

### Example: UC-03 - Create User Account

**Source Code Reference**: `controllers/user_account_controller.py` lines 26-85

```
Use Case ID: UC-03
Use Case Name: Create User Account
Actor: User Admin
Description: User Admin creates a new user account in the system

Preconditions:
- User Admin is logged in
- At least one active user profile exists

Postconditions:
- New user account is created and stored in database
- User account is associated with a user profile
- Password is securely hashed

Main Flow:
1. User Admin selects "Create User Account" option
2. System displays list of available user profiles
3. User Admin enters username, email, password, first name, last name, phone number (optional), and selects user profile
4. System validates that username is unique
5. System validates that email is unique
6. System validates that selected user profile exists and is active
7. System hashes the password using bcrypt
8. System creates new UserAccount entity
9. System saves UserAccount to database
10. System displays success message with account details

Alternative Flows:
3a. Username already exists
    3a.1. System displays error message "Username already exists"
    3a.2. Return to step 3

3b. Email already exists
    3b.1. System displays error message "Email already exists"
    3b.2. Return to step 3

3c. Invalid or inactive user profile selected
    3c.1. System displays error message "Invalid or inactive user profile"
    3c.2. Return to step 3

3d. Database error occurs
    3d.1. System rolls back transaction
    3d.2. System displays error message
    3d.3. Return to step 3
```

**Where to find the information:**
- Main Flow: Read `UserAccountController.create_user_account()` method
- Alternative Flows: Look at error handling in the same method
- Preconditions/Postconditions: Infer from the code logic

---

## 2️⃣ Use Case Diagram

### What to include:
- **Actor**: User Admin (stick figure)
- **System Boundary**: CSR Volunteering System (box)
- **Use Cases**: (ovals inside the box)
  1. Login
  2. Logout
  3. Create User Account
  4. View User Account
  5. Update User Account
  6. Suspend User Account
  7. Search User Accounts
  8. Create User Profile
  9. View User Profile
  10. Update User Profile
  11. Suspend User Profile
  12. Search User Profiles

### Relationships:
- All use cases are connected to User Admin actor
- **<<include>>** relationship: 
  - Create/Update/Suspend User Account → View User Account (to verify changes)
  - Create User Account → Search User Profiles (to select profile)

### Source:
- Use cases are the 12 user stories
- Actor is "User Admin"
- System boundary is the application

---

## 3️⃣ BCE Diagram

### Components:

**BOUNDARY (B)**
```
┌─────────────────────────┐
│  UserAdminBoundary      │
│  ───────────────────    │
│  + display_menu()       │
│  + handle_login()       │
│  + handle_logout()      │
│  + handle_create_user   │
│    _account()           │
│  + handle_view_user     │
│    _account()           │
│  + ...                  │
└─────────────────────────┘
```

**CONTROL (C)**
```
┌───────────────────────────┐
│ AuthenticationController  │
│ ────────────────────────  │
│ + login()                 │
│ + logout()                │
│ + get_current_user()      │
│ + is_logged_in()          │
└───────────────────────────┘

┌───────────────────────────┐
│ UserAccountController     │
│ ────────────────────────  │
│ + create_user_account()   │
│ + view_user_account()     │
│ + update_user_account()   │
│ + suspend_user_account()  │
│ + searchUserAccount()  │
└───────────────────────────┘

┌───────────────────────────┐
│ UserProfileController     │
│ ────────────────────────  │
│ + create_user_profile()   │
│ + view_user_profile()     │
│ + update_user_profile()   │
│ + suspend_user_profile()  │
│ + search_user_profiles()  │
└───────────────────────────┘
```

**ENTITY (E)**
```
┌─────────────────────┐
│  UserAccount        │
│  ──────────────     │
│  - id               │
│  - username         │
│  - email            │
│  - password_hash    │
│  - first_name       │
│  - last_name        │
│  - phone_number     │
│  - user_profile_id  │
│  - is_active        │
│  - created_at       │
│  - updated_at       │
│  ──────────────     │
│  + to_dict()        │
└─────────────────────┘

┌─────────────────────┐
│  UserProfile        │
│  ──────────────     │
│  - id               │
│  - profile_name     │
│  - description      │
│  - is_active        │
│  ──────────────     │
│  + to_dict()        │
└─────────────────────┘
```

**Relationships:**
- Boundary → Control (calls methods)
- Control → Entity (CRUD operations)
- UserAccount → UserProfile (many-to-one relationship)

### Source:
- Look at the folder structure: `boundaries/`, `controllers/`, `entities/`
- Methods are from the class definitions
- Attributes are from entity classes

---

## 4️⃣ Sequence Diagram

### Example: Create User Account (UC-03)

```
Participants:
- User Admin (Actor)
- UserAdminBoundary
- UserProfileController
- UserAccountController
- UserAccount (Entity)
- Database

Sequence:
1. User Admin → UserAdminBoundary: handle_create_user_account()
2. UserAdminBoundary → UserProfileController: get_active_user_profiles()
3. UserProfileController → Database: SELECT * FROM user_profiles WHERE is_active=1
4. Database → UserProfileController: [list of profiles]
5. UserProfileController → UserAdminBoundary: [list of profiles]
6. UserAdminBoundary → User Admin: display profiles
7. User Admin → UserAdminBoundary: enter user details
8. UserAdminBoundary → UserAccountController: create_user_account(username, email, ...)
9. UserAccountController → Database: SELECT FROM user_accounts WHERE username=?
10. Database → UserAccountController: [empty result]
11. UserAccountController → Database: SELECT FROM user_accounts WHERE email=?
12. Database → UserAccountController: [empty result]
13. UserAccountController → UserAccountController: bcrypt.hashpw(password)
14. UserAccountController → UserAccount: new UserAccount(...)
15. UserAccount → UserAccountController: [new user object]
16. UserAccountController → Database: INSERT INTO user_accounts
17. Database → UserAccountController: [success]
18. UserAccountController → UserAdminBoundary: (True, "success", user)
19. UserAdminBoundary → User Admin: display success message
```

### How to extract:
1. Start at boundary method (e.g., `handle_create_user_account()`)
2. Follow each method call in the code
3. Note each database query (SQLAlchemy operations)
4. Track the return values
5. End when control returns to the user

**Tip**: Read the method implementation line by line and trace the flow!

---

## 5️⃣ Sequence Diagram for Error Cases

### Example: Create User Account - Username Already Exists

```
1. User Admin → UserAdminBoundary: handle_create_user_account()
2. UserAdminBoundary → UserAdminBoundary: display profile list
3. User Admin → UserAdminBoundary: enter details (username="admin")
4. UserAdminBoundary → UserAccountController: create_user_account("admin", ...)
5. UserAccountController → Database: SELECT FROM user_accounts WHERE username="admin"
6. Database → UserAccountController: [existing user found]
7. UserAccountController → UserAdminBoundary: (False, "Username already exists", None)
8. UserAdminBoundary → User Admin: display error message
```

---

## 6️⃣ Class Diagram

### Classes to include:

**Entity Classes:**
```
┌─────────────────────────────────────────┐
│          UserProfile                     │
├─────────────────────────────────────────┤
│ - id: Integer                           │
│ - profile_name: String(100)             │
│ - description: Text                     │
│ - is_active: Boolean                    │
├─────────────────────────────────────────┤
│ + __init__()                            │
│ + to_dict(): dict                       │
│ + __repr__(): str                       │
└─────────────────────────────────────────┘
        ▲
        │
        │ 1
        │
        │ n
┌─────────────────────────────────────────┐
│          UserAccount                     │
├─────────────────────────────────────────┤
│ - id: Integer                           │
│ - username: String(50)                  │
│ - email: String(100)                    │
│ - password_hash: String(255)            │
│ - first_name: String(50)                │
│ - last_name: String(50)                 │
│ - phone_number: String(20)              │
│ - user_profile_id: Integer (FK)         │
│ - is_active: Boolean                    │
│ - created_at: DateTime                  │
│ - updated_at: DateTime                  │
├─────────────────────────────────────────┤
│ + __init__()                            │
│ + to_dict(): dict                       │
│ + __repr__(): str                       │
└─────────────────────────────────────────┘
```

**Controller Classes:**
```
┌────────────────────────────────────────────────────┐
│      AuthenticationController                      │
├────────────────────────────────────────────────────┤
│ - session: Session                                 │
│ - current_user: UserAccount                        │
├────────────────────────────────────────────────────┤
│ + login(username: str, password: str):             │
│     (bool, str, UserAccount)                       │
│ + logout(): (bool, str)                            │
│ + get_current_user(): UserAccount                  │
│ + is_logged_in(): bool                             │
│ + has_profile(profile_name: str): bool             │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│      UserAccountController                         │
├────────────────────────────────────────────────────┤
│ - session: Session                                 │
├────────────────────────────────────────────────────┤
│ + create_user_account(...): (bool, str, UserAccount)│
│ + view_user_account(user_id: int):                 │
│     (bool, str, UserAccount)                       │
│ + update_user_account(...): (bool, str, UserAccount)│
│ + suspend_user_account(user_id: int): (bool, str)  │
│ + activate_user_account(user_id: int): (bool, str) │
│ + searchUserAccount(...): (bool, str, list)     │
│ + get_all_user_accounts(): list                    │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│      UserProfileController                         │
├────────────────────────────────────────────────────┤
│ - session: Session                                 │
├────────────────────────────────────────────────────┤
│ + create_user_profile(...):                        │
│     (bool, str, UserProfile)                       │
│ + view_user_profile(profile_id: int):              │
│     (bool, str, UserProfile)                       │
│ + update_user_profile(...):                        │
│     (bool, str, UserProfile)                       │
│ + suspend_user_profile(profile_id: int):           │
│     (bool, str)                                    │
│ + activate_user_profile(profile_id: int):          │
│     (bool, str)                                    │
│ + search_user_profiles(...): (bool, str, list)     │
│ + get_all_user_profiles(): list                    │
│ + get_active_user_profiles(): list                 │
└────────────────────────────────────────────────────┘
```

**Boundary Class:**
```
┌────────────────────────────────────────────────────┐
│      UserAdminBoundary                             │
├────────────────────────────────────────────────────┤
│ - auth_controller: AuthenticationController        │
│ - account_controller: UserAccountController        │
│ - profile_controller: UserProfileController        │
├────────────────────────────────────────────────────┤
│ + display_menu(): void                             │
│ + run(): void                                      │
│ + handle_login(): void                             │
│ + handle_logout(): void                            │
│ + handle_create_user_account(): void               │
│ + handle_view_user_account(): void                 │
│ + handle_update_user_account(): void               │
│ + handle_suspend_activate_user_account(): void     │
│ + handle_searchUserAccount(): void              │
│ + handle_list_all_user_accounts(): void            │
│ + handle_create_user_profile(): void               │
│ + handle_view_user_profile(): void                 │
│ + handle_update_user_profile(): void               │
│ + handle_suspend_activate_user_profile(): void     │
│ + handle_search_user_profiles(): void              │
│ + handle_list_all_user_profiles(): void            │
│ + display_user_account(user: UserAccount): void    │
│ + display_user_accounts_table(users: list): void   │
│ + display_user_profile(profile: UserProfile): void │
│ + display_user_profiles_table(profiles: list): void│
└────────────────────────────────────────────────────┘
```

### Relationships:
- UserAdminBoundary **uses** → AuthenticationController
- UserAdminBoundary **uses** → UserAccountController
- UserAdminBoundary **uses** → UserProfileController
- Controllers **access** → Entity classes
- UserAccount **references** → UserProfile (FK relationship)

---

## 7️⃣ Wireframes

### Based on CLI screens, create UI mockups

**Example: Login Screen**
```
Source: boundaries/user_admin_boundary.py - handle_login()

CLI Version:
--- LOGIN ---
Username: _____
Password: _____

Web Mockup:
┌─────────────────────────────────────┐
│  CSR Volunteering System            │
│  ─────────────────────────────────  │
│                                     │
│  Login                              │
│                                     │
│  Username: [________________]       │
│                                     │
│  Password: [________________]       │
│                                     │
│  [Login Button]                     │
│                                     │
└─────────────────────────────────────┘
```

**Example: Create User Account Screen**
```
Source: boundaries/user_admin_boundary.py - handle_create_user_account()

┌──────────────────────────────────────────────────┐
│  Create User Account                              │
│  ──────────────────────────────────────────────  │
│                                                   │
│  Username:     [___________________]              │
│  Email:        [___________________]              │
│  Password:     [___________________]              │
│  First Name:   [___________________]              │
│  Last Name:    [___________________]              │
│  Phone:        [___________________]              │
│                                                   │
│  User Profile: [▼ Select Profile   ]             │
│                 ☐ User Admin                      │
│                 ☐ Platform Manager                │
│                 ☐ CSR Rep                         │
│                 ☑ PIN                             │
│                                                   │
│  [Create]  [Cancel]                               │
│                                                   │
└──────────────────────────────────────────────────┘
```

**Example: Search Results Screen**
```
Source: boundaries/user_admin_boundary.py - display_user_accounts_table()

┌──────────────────────────────────────────────────────────────────┐
│  Search User Accounts                                             │
│  ──────────────────────────────────────────────────────────────  │
│                                                                   │
│  Keyword: [__________]  Profile: [▼ All]  Status: [▼ All]       │
│                                                [Search]           │
│                                                                   │
│  Results: 15 user accounts found                                 │
│  ┌────┬────────────┬─────────────────────┬──────────┬─────────┐ │
│  │ ID │ Username   │ Email               │ Profile  │ Status  │ │
│  ├────┼────────────┼─────────────────────┼──────────┼─────────┤ │
│  │ 1  │ admin      │ admin@csr.com       │ Admin    │ Active  │ │
│  │ 2  │ john_doe   │ john@company.com    │ CSR Rep  │ Active  │ │
│  │ 3  │ jane_smith │ jane@email.com      │ PIN      │ Suspend │ │
│  │... │ ...        │ ...                 │ ...      │ ...     │ │
│  └────┴────────────┴─────────────────────┴──────────┴─────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📊 Quick Reference Table

| Artifact | Source File | Key Information |
|----------|-------------|-----------------|
| Use Case Descriptions | Controller methods | Read method docstrings and code logic |
| Use Case Diagram | All 12 user stories | Actor = User Admin, Use cases = Stories |
| BCE Diagram | Folder structure | boundaries/, controllers/, entities/ |
| Sequence Diagrams | Controller methods | Trace method calls line by line |
| Class Diagram | Entity & Controller classes | Class definitions and attributes |
| Wireframes | Boundary class | CLI prompts → UI mockups |
| Database Schema | Entity classes | SQLAlchemy Column definitions |

---

## Tips

1. **Use Case Descriptions**: Each controller method has comments linking to the user story number
2. **Sequence Diagrams**: Use debugger or print statements to trace exact flow
3. **Class Diagram**: Copy class structure directly from code
4. **Wireframes**: The CLI is a prototype - convert to graphical UI
5. **Consistency**: Ensure names in diagrams match code exactly

---

## Validation Checklist

✅ All 12 user stories have use case descriptions  
✅ Use case diagram includes all 12 use cases  
✅ BCE diagram shows all boundary, control, and entity classes  
✅ At least one sequence diagram per user story  
✅ Class diagram includes all attributes and methods  
✅ Wireframes cover main user interactions  
✅ Database schema matches entity classes  
✅ Design artifacts are consistent with code  

---

**Remember**: The code is your "source of truth". Your diagrams should accurately reflect what the code does!

