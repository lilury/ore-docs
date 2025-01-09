---
sidebar_position: 3
---

# `Creator[T]` interface

```go
type Creator[T] interface {
   New(ctx context.Context) (T, context.Context)
}
```

In order to instruct Ore to use `Creator[T]` to build and initialize new instances of service `T`,
make sure that your service's `struct` implements `Creator[T]` :

## Example Usage

```go
type Counter interface {
    GetCount() int
    AddOne()
}

type simpleCounterImpl struct {
    counter int
}

// implementing Creator[T]
func (c *simpleCounterImpl) New(ctx context.Context) (Counter, context.Context) {
   return &simpleCounterImpl{
      counter: 0
   }, ctx
}

func (c *simpleCounterImpl) AddOne() {
    c.counter += 1
}

func (c *simpleCounterImpl) GetCount() int {
    return c.counter
}
```

## Registering `Creator[T]` service

```go
ore.RegisterCreator[Counter](ore.Scoped, &simpleCounterImpl{})
```

Note that here we used `ore.RegisterCreator` instead of `ore.RegisterFunc`.

## Resolving `Creator[T]` service

```go
counterService, ctx := ore.Get[Counter](context.Background())
```

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

// implementing `Creator[T]`
func (m *smtpEmailProvider) New(ctx context.Context) (EmailProvider, context.Context) {
   return &smtpEmailProvider{
      host: "smtp.gmail.com",
      port: 587,
      username: "someemailaddress@gmail.com",
      password: "somepasswordhere",
   }, ctx
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

// implementing `Creator[T]`
func (m *smsGatewayProvider) New(ctx context.Context) (SmsProvider, context.Context) {
   return &smsGatewayProvider{
        apiUrl: "https://gateway.somesmsprovider.com",
        apiKey: "1f7b3bb5-ce07-4119-864d-0995f6f89515",
    }, ctx
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

// implementing `Creator[T]`
func (un *userNotifier) New(ctx context.Context) (NotificationService, context.Context) {
   emailProviderService, ctx := ore.Get[EmailProvider](ctx)
   smsProviderService, ctx := ore.Get[SmsProvider](ctx)
   
   return &userNotifier{
      emailProvider: emailProviderService,
      smsProvider: smsProviderService,
    }, ctx
}

func (un *userNotifier) NotifyUser(email string, phone string, message string) error {
    err := un.emailProvider.SendEmail(email, "Notification", message)
    err = un.smsProvider.SendSms(phone, message)
    
    return err
}
```

## Register Services via `Creator[T]` (RegisterCreator)

registering `EmailProvider` service:

```go
ore.RegisterCreator[EmailProvider](ore.Scoped, &smtpEmailProvider{})
```

registering `SmsProvider` service:

```go
ore.RegisterCreator[SmsProvider](ore.Scoped, &smsGatewayProvider{})
```

registering `NotificationService` service:

```go
ore.RegisterCreator[NotificationService](ore.Scoped, &userNotifier{})
```



