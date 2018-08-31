1. To get barong access token:

```bash
curl -X POST -d '{"email":"...", "password": "...", "application_id": "...", "otp_code":"..."}' -H "Content-Type: application/json" http://localhost:3000/api/v1/sessions
```

Parameters:

```yaml
'totp_code': code from Google Authenticator
```

 To get application_id:

in barong rails c -> 

```ruby
Doorkeeper::Application.first.uid
```

2. To create api_key you should make this request to barong:

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: Bearer {jwt_access_token}" -d '{"public_key":"", "totp_code":"...", "scopes":"peatio"}' https://localhost:3000/api/v1/api_keys
```

3. To get public_key you can generate keypair using and set public key to previous request:

```bash
ruby -e "require 'openssl'; require 'base64'; OpenSSL::PKey::RSA.generate(2048).tap { |p| puts '', 'PRIVATE RSA KEY (URL-safe Base64 encoded, PEM):', '', Base64.urlsafe_encode64(p.to_pem), '', 'PUBLIC RSA KEY (URL-safe Base64 encoded, PEM):', '', Base64.urlsafe_encode64(p.public_key.to_pem) }"
```

To get api_keys you should make this request to barong:

```bash
curl -X GET -H "Authorization: Bearer {jwt_access_token}" http://localhost:3000/api/v1/api_keys?totp_code=string
```

4. To generate JWT_api_token you need to encode private key and your valid payload with following ruby code.

```ruby
require 'openssl'
require 'base64'
require 'json'
require 'securerandom'
require 'jwt'
require 'active_support/time'


secret_key = OpenSSL::PKey.read(Base64.urlsafe_decode64('private_key'))
payload = {
  iat: Time.current.to_i,
  exp: 20.minutes.from_now.to_i,
  sub: 'api_key_jwt',
  iss: 'external',
  jti: SecureRandom.hex(12).upcase
}
puts jwt_token = JWT.encode(payload, secret_key, 'RS256')
```

To get JWT_api_token you make this request to barong:

```bash
curl -X POST -H 'Content-Type: application/json' -d '{"kid":"...", "jwt_token":"..."}' http://localhost:3000/api/v1/sessions/generate_jwt
```

Parameters:

```yml
'kid': uid of the api_key
'jwt_token': token from ruby script
```

5. To create order you should make this request to peatio: 

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: Bearer {jwt_api_token}" -d '{"market":"", "side":"", "volume":"", "price":""}' http://localhost:3000/api/v2/orders
```

Parameters:

```yaml
'jwt_api_token': token from previous step
```

Example for data:

```bash
-d '{"market":"btcusd", "side":"buy", "volume":"12.5", "price":"2014"}'
```