# Taxi Backend

Foydalanuvchi balansiga to'lovlarni qabul qilish uchun mo'ljallangan backend loyiha. Payme, Click, Paynet va Uzum to'lov tizimlari orqali foydalanuvchilar o'z hamyonlarini to'ldirishlari mumkin.

## O'rnatish

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

## Sozlash

`.env.sample` faylini `.env` ga nusxalang va to'lov tizimlaridan olgan kalitlaringizni kiriting:

```bash
cp .env.sample .env
```

## API Hujjatlari

Quyida mavjud API'lar va ulardan foydalanish uchun curl misollar keltirilgan.

### Authentication

**1. Registratsiya**

Yangi foydalanuvchi yaratish (va uning hamyonini avtomatik yaratish).

*   **URL:** `/api/auth/register/`
*   **Method:** `POST`

```bash
curl -X POST http://127.0.0.1:8000/api/auth/register/ \
     -H "Content-Type: application/json" \
     -d '{
           "username": "new_user",
           "password": "secure_password",
           "first_name": "John",
           "last_name": "Doe"
         }'
```

**2. Login (Token olish)**

Foydalanuvchi tizimga kirishi va JWT tokenlarini (access va refresh) olishi uchun.

*   **URL:** `/api/auth/login/`
*   **Method:** `POST`

```bash
curl -X POST http://127.0.0.1:8000/api/auth/login/ \
     -H "Content-Type: application/json" \
     -d '{
           "username": "your_username",
           "password": "your_password"
         }'
```

**Response:**
```json
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

**2. Tokenni yangilash (Refresh Token)**

Access token muddati tugagandan so'ng yangi access token olish uchun.

*   **URL:** `/api/auth/token/refresh/`
*   **Method:** `POST`

```bash
curl -X POST http://127.0.0.1:8000/api/auth/token/refresh/ \
     -H "Content-Type: application/json" \
     -d '{
           "refresh": "YOUR_REFRESH_TOKEN"
         }'
```


**4. Logout**

Tokenni "qora ro'yxat"ga kiritish orqali tizimdan chiqish (ixtiyoriy, token muddati tugaguncha ishlaydi, agar blacklist ishlatilmasa).

*   **URL:** `/api/auth/logout/`
*   **Method:** `POST`

```bash
curl -X POST http://127.0.0.1:8000/api/auth/logout/ \
     -H "Content-Type: application/json" \
     -d '{
           "refresh": "YOUR_REFRESH_TOKEN"
         }'
```

**5. Profil**

Foydalanuvchi ma'lumotlari va hamyon balansini olish.

*   **URL:** `/api/auth/profile/`
*   **Method:** `GET`
*   **Headers:** `Authorization: Bearer <ACCESS_TOKEN>`

```bash
curl -X GET http://127.0.0.1:8000/api/auth/profile/ \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Response:**
```json
{
    "id": 1,
    "username": "new_user",
    "first_name": "John",
    "last_name": "Doe",
    "wallet": {
        "id": 1,
        "balance": "0.00"
    }
}
```

### Payments

**1. Balansni to'ldirish**

Payme yoki Click orqali balansni to'ldirish uchun to'lov linkini yaratish. Bu endpoint **autentifikatsiya talab qiladi**.

*   **URL:** `/api/payments/top-up/`
*   **Method:** `POST`
*   **Headers:** `Authorization: Bearer <ACCESS_TOKEN>`

```bash
curl -X POST http://127.0.0.1:8000/api/payments/top-up/ \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
           "amount": 50000,
           "provider": "payme"
         }'
```
*Provider qiymatlari: `payme`, `click`, `uzum`, `paynet`. *

**Response (Payme):**
```json
{
    "payment_url": "https://checkout.paycom.uz/..."
}
```

**Response (Click):**
```json
{
    "payment_url": "https://my.click.uz/services/pay?service_id=123&merchant_id=123&amount=1000&transaction_param=2"
}
```

**Response (Uzum):**
```json
{
    "payment_url": "https://www.uzumbank.uz/open-service?serviceId=123&order_id=2&amount=100000&redirectUrl=https://example.com"
}
```

**Response (Paynet):**
```json
{
    "payment_url": "https://app.paynet.uz/?m=675&c=2"
}
```
