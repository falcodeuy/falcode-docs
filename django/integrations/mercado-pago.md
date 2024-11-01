---
layout: default
title: Mercado pago
parent: Integrations
grand_parent: Django
nav_order: 1
---

# Mercado Pago Integration

## Initial Instructions

### How to Create a Development App Using the Checkout API

1. Visit the [Mercado Pago Developer Portal](https://www.mercadopago.com/developers).
2. Create an account or log in if you already have one.
3. Navigate to the **My Apps** section and click **Create App**.
4. Fill in the necessary details such as **App Name** and **Description**.
5. Make sure to select **Checkout API** as the integration type.

### How to Specify the Callback URL

- Set the **Redirect URI** as required for OAuth flows.
- For local development, it's highly recommended to use a tool like [ngrok](https://ngrok.com/) to expose your local server over HTTPS. This is required as Mercado Pago only accepts URLs with HTTPS.
- Example: `https://e4b7-2800-a4-93e-a100-65dd-80de-e655-62c1.ngrok-free.app/mercado-pago/oauth_callback`

### How to Fill in the Following Data

In your Django settings or environment variables, fill in the following keys:

- **MERCADO_PAGO_CLIENT_ID**: You can find this in the **My Apps** section of your Mercado Pago account after creating an app.
- **MERCADO_PAGO_CLIENT_SECRET**: This is also available in the **My Apps** section, under the app details.
- **MERCADO_PAGO_REDIRECT_URI**: This is the URL where Mercado Pago will redirect after the OAuth flow. Use the ngrok URL for local development.
- **MERCADO_PAGO_PUBLIC_KEY**: Available in the **Credentials** section of your app in Mercado Pago.
- **MERCADO_PAGO_ACCOUNT_ID**: This is your **Collector ID**, which can be found in the **Account Settings** section of your Mercado Pago account.

```python
MERCADO_PAGO_CLIENT_ID = '4064746753721270'
MERCADO_PAGO_CLIENT_SECRET = 'your_client_secret'
MERCADO_PAGO_REDIRECT_URI = 'https://e4b7-2800-a4-93e-a100-65dd-80de-e655-62c1.ngrok-free.app/mercado-pago/oauth_callback'
MERCADO_PAGO_PUBLIC_KEY = 'your_public_key'
MERCADO_PAGO_ACCOUNT_ID = 'your_collector_id'
```

## Base Helper Class

We have a helper class for integrating with Mercado Pago. Assuming the app is named `payment`, add the following class in `payments/services/MercadoPago.py`. The class can be placed anywhere; this is just a practice given by our company.

```python
class MercadoPagoBase(ABC):

    @abstractmethod
    def generate_token(self):
        pass

    @staticmethod
    def register_application_link():
        token = self.generate_token()
        encoded_redirect_uri = quote(f"{settings.MERCADO_PAGO_REDIRECT_URI}", safe='')
        auth_url = (
            f"https://auth.mercadopago.com/authorization"
            f"?client_id={settings.MERCADO_PAGO_CLIENT_ID}"
            f"&response_type=code"
            f"&platform_id=mp"
            f"&redirect_uri={encoded_redirect_uri}"
            f"&state={token}"
        )
        return auth_url

    @staticmethod
    def register_application_callback(code):
        data = {
            'grant_type': 'authorization_code',
            'client_id': settings.MERCADO_PAGO_CLIENT_ID,
            'client_secret': settings.MERCADO_PAGO_CLIENT_SECRET,
            'code': code,
            'redirect_uri': settings.MERCADO_PAGO_REDIRECT_URI,
        }
        response = requests.post('https://api.mercadopago.com/oauth/token', data=data)

        if response.status_code == 200:
            tokens = response.json()
            mp_account, _ = MercadoPagoAccount.objects.get_or_create()
            mp_account.mercado_pago_user_id = tokens.get('user_id')
            mp_account.access_token = tokens.get('access_token')
            mp_account.refresh_token = tokens.get('refresh_token')
            mp_account.expires_at = timezone.now() + timedelta(seconds=tokens.get('expires_in', 0))
            mp_account.save()
            return mp_account

    @staticmethod
    def create_split_payment(mp_account, amount, payment_method, buyer_email, fee_percentage):
        """Create a new split payment on the specified account with the fee applied to the principal account"""
        payment_data = {
            "transaction_amount": amount,
            "description": "Service Payment",
            "payment_method_id": payment_method,
            "payer": {
                "email": buyer_email
            },
            "application_fee": amount * fee_percentage,
            "collector_id": settings.MERCADO_PAGO_ACCOUNT_ID
        }

        headers = {
            "Authorization": f"Bearer {mp_account.access_token}",
            "Content-Type": "application/json",
        }
        response = requests.post('https://api.mercadopago.com/v1/payments', headers=headers, json=payment_data)
        if response.status_code == 201:
            return response.json()

    @staticmethod
    def create_new_payment(amount):
        """Create a new payment for the principal account"""
        preference_data = {
            "items": [
                {
                    "title": "Monthly Subscription",
                    "quantity": 1,
                    "unit_price": amount,
                }
            ],
            "auto_return": "approved",
        }

        headers = {
            "Authorization": f"Bearer {settings.MERCADO_PAGO_ACCESS_TOKEN}",
            "Content-Type": "application/json",
        }

        response = requests.post('https://api.mercadopago.com/checkout/preferences', headers=headers,
                                 json=preference_data)
        if response.status_code == 201:
            return response.json()
```

Then create a class that inherits from this one and specifies the following method to get the token, for example:

```python
class MercadoPagoService(MercadoPagoBase):
    @staticmethod
    def generate_token():
        user = get_user_model().objects.get(name__icontains='Vendedor')
        token = VerificationToken.generate_token_for_user(user, VerificationToken.Type.MERCADO_PAGO)
        return token.token
```

You will also need to add the following model, which you can extend as needed:

```python
class MercadoPagoAccount(models.Model):
    mercado_pago_user_id = models.CharField(max_length=255, null=True, blank=True)
    access_token = models.CharField(max_length=255, null=True, blank=True)
    refresh_token = models.CharField(max_length=255, null=True, blank=True)
    expires_at = models.DateTimeField(null=True, blank=True)
```

Then declare the relationship to this new Mercado Pago account model in the appropriate model.

With this, you are ready to start integrating Mercado Pago using the specified classes.

## Integrating Payments

### Split Payments Integration

To integrate split payments, refer to the following link from the Mercado Pago documentation: [Split Payments Integration](https://www.mercadopago.com.uy/developers/es/docs/split-payments/integration-configuration/create-configuration#bookmark_crear_aplicaci%C3%B3n)

The split payment feature allows you to create payments where part of the payment is sent to another account, and the rest is kept by your collector account.

## Step-by-Step Usage Example with Views

### Step 1: User Initiates Application Registration

In your web application, provide a button that users can click to register their Mercado Pago account. When they click this button, make a request to the following view:

```python
@action(detail=False, methods=['post'])
def register_application(self, request):
    auth_url = MercadoPagoService.register_application_link()
    return Response({'auth_url': auth_url})
```

- **User Interaction**: The user clicks a button to connect their Mercado Pago account.
- **Backend Action**: This calls `register_application_link()`, which returns an `auth_url` that redirects the user to Mercado Pago for authorization.

### Step 2: User Authorizes the Application

The user is redirected to Mercado Pago where they authorize the application to access their data.

- **User Interaction**: The user approves the requested permissions on the Mercado Pago authorization page.

### Step 3: Handling the OAuth Callback

After the user authorizes the application, Mercado Pago redirects them to the callback URL (`MERCADO_PAGO_REDIRECT_URI`). You need a view to handle this callback:

```python
@action(detail=False, methods=['get'])
def oauth_callback(self, request):
    serializer = CallBackSerializer(data=request.query_params)
    serializer.is_valid(raise_exception=True)
    data = serializer.validated_data

    token = data.get('state')
    code = data.get('code')

    try:
        verification_token = VerificationToken.objects.get(token=token,
                                                           token_type=VerificationToken.Type.MERCADO_PAGO,
                                                           expires_at__gte=timezone.now())
        user = verification_token.user
    except VerificationToken.DoesNotExist:
        return Response({'error': 'Invalid or expired token'}, status=status.HTTP_400_BAD_REQUEST)

    account = MercadoPagoService.register_application_callback(code, user)

    if isinstance(account, MercadoPagoAccount):
        return Response({'message': 'Application registered successfully'})
    else:
        return Response({'error': account.get('error', 'Failed to obtain access token')}, status=status.HTTP_400_BAD_REQUEST)
```

- **User Interaction**: No direct interaction; this happens after the user approves the authorization on Mercado Pago.
- **Backend Action**: This view handles the callback, validates the `code` and `state`, and saves the tokens to `MercadoPagoAccount` for the user.

### Step 4: Create a Payment for a Registered Company

Once the user's Mercado Pago account is registered, they can initiate a payment. Use the following view to create a split payment:

```python
@action(detail=False, methods=['post'])
def create_payment_for_company(self, request):
    user = request.user

    serializer = SplitPaymentSerializer(data=request.data, context={'user': user})
    serializer.is_valid(raise_exception=True)
    data = serializer.validated_data

    payment_info = MercadoPagoService.create_split_payment(
        data.get('company').mercado_pago_account,
        data.get('amount'),
        data.get('payment_method'),
        user.email,
        3
    )

    if 'error' in payment_info:
        return Response({'error': payment_info['error']}, status=status.HTTP_400_BAD_REQUEST)
    return Response({'payment_info': payment_info})
```

- **User Interaction**: The user requests to create a payment.
- **Backend Action**: This view calls `create_split_payment()`, which handles the logic to split the payment and returns payment details.
