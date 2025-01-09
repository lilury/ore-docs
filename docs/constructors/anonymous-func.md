---
sidebar_position: 2
---

# Anonymous Functions

In Go, **anonymous functions** are functions defined without a name. They are often used as inline functions, assigned
to variables, or passed as arguments to other functions. Anonymous functions are useful when you need a quick,
short-lived function without the need to define it separately.

### Syntax

An anonymous function is defined using the `func` keyword, followed by its parameters and body, without a name:

```go
func(parameter_list) return_type {
    // function body
}
```

### Example Usage

1. **Assigning to a Variable**:
   ```go
   add := func(a int, b int) int {
       return a + b
   }
   fmt.Println(add(3, 4)) // Output: 7
   ```

2. **Inline Execution**:
   ```go
   result := func(a, b int) int {
       return a * b
   }(3, 5)
   fmt.Println(result) // Output: 15
   ```

3. **Passing as an Argument**:
   ```go
   func apply(f func(int) int, value int) int {
       return f(value)
   }

   square := func(x int) int { return x * x }
   fmt.Println(apply(square, 4)) // Output: 16
   ```

### Characteristics

- **No Name**: They are defined without a function name.
- **Closures**: Anonymous functions can capture and use variables from their surrounding scope, making them closures.
- **Short-Lived**: Typically used for small, localized tasks.

### Common Use Cases

- Callback functions.
- Short-lived logic within higher-order functions like `sort.Slice`.
- Defining behavior inline without polluting the global namespace.

## Sample Service

let's define a `NotificationService` that sends a notification to a user via Email and SMS:

```go
// EmailProvider defines the interface for sending emails.
type EmailProvider interface {
    SendEmail(to string, subject string, body string) error
}

// SmsProvider defines the interface for sending SMS messages.
type SmsProvider interface {
    SendSms(to string, message string) error
}

// NotificationService defines the interface for notifying users.
type NotificationService interface {
    NotifyUser(email string, phone string, message string) error
}
```

note that `NotificationService` logically depends on `EmailProvider` and `SmsProvider`

```go
type smtpEmailProvider struct {
    host string
    port int
    username string
    password string
}

func (m *smtpEmailProvider) connect() (SMTPConnection, error) {
    // connect to SMTP server
}

func (m *smtpEmailProvider) send(conn SMTPConnection, to string, subject string, body string) error {
    // send an email via SMTP server connection
    conn.Send(to, subject, body)
}

func (m *smtpEmailProvider) SendEmail(to string, subject string, body string) error {
    // connect to SMTP server
    conn, err := m.connect(m.host, m.port, m.username, m.password)
    err = m.send(conn, to, subject, body)
    
    return err
}
```

```go
type smsGatewayProvider struct {
    apiUrl string
    apiKey string
}

func (m *smsGatewayProvider) SendSms(to string, message string) error {
    someSmsSender.Send(m.apiUrl, m.apiKey, to, message)
    
    return err
}
```

```go
type userNotifier struct {
    emailProvider EmailProvider
    smsProvider SmsProvider
}

func (un *userNotifier) NotifyUser(email string, phone string, message string) error {
    err := un.emailProvider.SendEmail(email, "Notification", message)
    err = un.smsProvider.SendSms(phone, message)
    
    return err
}
```

## Register Services via anonymous funcs (RegisterFunc)

registering `EmailProvider` service:

```go
ore.RegisterFunc[EmailProvider](ore.Scoped, func(ctx context.Context) (EmailProvider, error) {
    return &smtpEmailProvider{
        host: "smtp.gmail.com",
        port: 587,
        username: "someemailaddress@gmail.com",
        password: "somepasswordhere",
    }, ctx
})
```

registering `SmsProvider` service:

```go
ore.RegisterFunc[SmsProvider](ore.Scoped, func(ctx context.Context) (SmsProvider, error) {
    return &smsGatewayProvider{
        apiUrl: "https://gateway.somesmsprovider.com",
        apiKey: "1f7b3bb5-ce07-4119-864d-0995f6f89515",
    }, ctx
})
```

registering `NotificationService` service:

```go
ore.RegisterFunc[NotificationService](ore.Scoped, func(ctx context.Context) (NotificationService, error) {
    emailProviderService, ctx := ore.Get[EmailProvider](ctx)
    smsProviderService, ctx := ore.Get[SmsProvider](ctx)
    
    return &userNotifier{
        emailProvider: emailProviderService,
        smsProvider: smsProviderService,
    }, ctx
})
```

Note that the scope (`ctx`) is well passed through all dependencies of the `NotificationService` chain; This is very
crucial for the scoped services to be resolved correctly.


