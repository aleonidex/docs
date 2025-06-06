---
title: How to get an IAM token for a service account in {{ iam-full-name }}
description: Follow this guide to get an IAM token for a service account.
---

# Getting an IAM token for a service account

## Selecting a method to get an IAM token {#choose-method}

There are multiple options you can use to get an [IAM token](../../concepts/authorization/iam-token.md) for the [service account](../../concepts/users/service-accounts.md):

![choose-method](../../../_assets/iam/choose-method.svg)

* [VM metadata service](../../../compute/operations/vm-connect/auth-inside-vm.md): Preferred method when using a [{{ compute-full-name }}](../../../compute/) VM instance.
* [Workload identity federation](../../concepts/workload-identity.md): If you want to authenticate to the {{ yandex-cloud }} API upon request from an external system compatible with the [OpenID Connect](https://openid.net/developers/how-connect-works/) (OIDC) protocol. For example, when integrating with GitHub, [{{ mgl-full-name }}](../../../managed-gitlab/), or a custom {{ k8s }} installation.
* [CLI](#via-cli): This is the easiest way if the external system is not OIDC-compatible but allows you to install the CLI.
* [JSON Web Token](#via-jwt): Allows you to control IAM token generation at every step.
* [Function](../../../functions/operations/function-sa.md): To get an IAM token from the function code in [{{ sf-name }}](../../../functions/).

{% include [iam-token-lifetime](../../../_includes/iam-token-lifetime.md) %}

## Get an IAM token using the CLI {#via-cli}

{% include [cli-set-sa-profile](../../../_includes/cli-set-sa-profile.md) %}

Now you can get an IAM token for your service account:


```
yc iam create-token
```



{% include [iam-token-usage](../../../_includes/iam-token-usage.md) %}

{% note tip %}

You can use the profile you created to perform CLI operations under your service account.

{% endnote %}

## Get an IAM token using a JWT {#via-jwt}

To get an IAM token, create a [JSON Web Token](https://tools.ietf.org/html/rfc7519) (JWT) and exchange it for an IAM token.

### Getting started {#before-you-begin}

1. [Find out the service account ID](../sa/get-id.md).
1. [Create the authorized keys](../authentication/manage-authorized-keys.md#create-authorized-key) required for generating a JWT. Save the public key ID.

### 1. To create a JWT {#jwt-create}

Create a JWT manually by following the instructions or use a library for your programming language.

{% note tip %}

On [jwt.io](https://jwt.io) you can view the list of libraries and try generating a token manually.

{% endnote %}

{% list tabs group=programming_language %}

- Guide {#instruction}

  Generate the parts that make up a JWT:
  * `header`: Base64Url-encoded JWT headers.
  * `payload`: Base64Url-encoded JWT Claims Set.
  * `signature`: Signature generated from parts of the header and payload.

  To create a JWT, join all parts using a period as the delimiter:

  ```
  header.payload.signature
  ```

  **1.1. Generating a header**

  A service account's JWT header must contain the following fields:
  * `typ`: Token type, the value is always `JWT`.
  * `alg`: Encryption algorithm. The only supported algorithm is [PS256](https://tools.ietf.org/html/rfc7518#section-3.5).
  * `kid`: ID of the public key obtained when [creating authorized keys](../authentication/manage-authorized-keys.md#create-authorized-key). The key must belong to the service account that the IAM token is requested for.

  Example:

  ```
  {
    "typ": "JWT",
    "alg": "PS256",
    "kid": "lfkoe35hsk58********"
  }
  ```

  Save the result as a Base64Url-encoded string.

  **1.2. Generating a payload**

  A service account's JWT payload must contain the following fields:
  * `iss`: ID of the service account whose key the JWT is signed with.
  * `aud`: Link by which an IAM token will be requested: `https://iam.{{ api-host }}/iam/v1/tokens`.
  * `iat`: JWT issue time in [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) format.
  * `exp`: JWT expiration time in Unix timestamp format. The expiration time must not exceed the issue time by more than one hour, i.e., `exp - iat ≤ 3600`.

  Example:

  ```
  {
    "iss": "ajepg0mjt06s********",
    "aud": "https://iam.{{ api-host }}/iam/v1/tokens",
    "iat": 1516239022,
    "exp": 1516240822
  }
  ```

  Save the result as a Base64Url-encoded string.

  **1.3. Generating a signature**

  Create a signature using the private key obtained when [creating authorized keys](../authentication/manage-authorized-keys.md#create-authorized-key). For the signature, use a string consisting of the header and payload separated by a period (`.`):

  ```
  header.payload
  ```

  The only supported algorithm is [PS256](https://tools.ietf.org/html/rfc7518#section-3.5).

  Save the result as a Base64Url-encoded string.

  If you generate a token using [jwt.io](https://jwt.io), note that `\n` in the key value must be replaced with line breaks.

- Python {#python}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [PyJWT](https://github.com/jpadilla/pyjwt/):
  - Tested for Python 3.13.0 and PyJWT 2.10.0.
  - Required data is read from the JSON file obtained when creating the authorized key.

  Install the `PyJWT` and `cryptography` modules to use `PS256` algorithm:
  
  ```bash
  pip3 install PyJWT
  pip3 install cryptography
  ```

  
  ```python
  import time
  import jwt
  import json

  key_path = '<JSON_file_with_keys>'

  # Reading a private key from a JSON file
  with open(key_path, 'r') as f:
    obj = f.read() 
    obj = json.loads(obj)
    private_key = obj['private_key']
    key_id = obj['id']
    service_account_id = obj['service_account_id']

  sa_key = {
      "id": key_id,
      "service_account_id": service_account_id,
      "private_key": private_key
  }

  def create_jwt():
      now = int(time.time())
      payload = {
              'aud': 'https://iam.api.cloud.yandex.net/iam/v1/tokens',
              'iss': service_account_id,
              'iat': now,
              'exp': now + 3600
          }

      # JWT generation.
      encoded_token = jwt.encode(
          payload,
          private_key,
          algorithm='PS256',
          headers={'kid': key_id}
      )

      print(encoded_token)

      return encoded_token

  create_jwt()
  ```


- Java {#java}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using the [JJWT](https://github.com/jwtk/jjwt), [Bouncy Castle](https://github.com/bcgit/bc-java), and [Jackson Databind](https://github.com/FasterXML/jackson-databind) libraries:
  - Verified for Java 21 and JJWT 0.12.5.
  - Required data is read from the JSON file obtained when creating the authorized key.

  ```java
  package com.mycompany.java.jwt;

  import com.fasterxml.jackson.databind.ObjectMapper;
  import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
  import io.jsonwebtoken.Jwts;
  import io.jsonwebtoken.SignatureAlgorithm;
  import org.bouncycastle.util.io.pem.PemObject;
  import org.bouncycastle.util.io.pem.PemReader;

  import java.io.StringReader;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.security.KeyFactory;
  import java.security.PrivateKey;
  import java.security.spec.PKCS8EncodedKeySpec;
  import java.time.Instant;
  import java.util.Date;

  public class JavaJwt {

      @JsonIgnoreProperties(ignoreUnknown = true)
      public static class KeyInfo {

          public String id;
          public String service_account_id;
          public String private_key;
      }

      public static void main(String[] args) throws Exception {

          String content = new String(Files.readAllBytes(Paths.get("<JSON_file_with_keys>")));
          KeyInfo keyInfo = (new ObjectMapper()).readValue(content, KeyInfo.class);

          String privateKeyString = keyInfo.private_key;
          String serviceAccountId = keyInfo.service_account_id;
          String keyId = keyInfo.id;

          PemObject privateKeyPem;
          try (PemReader reader = new PemReader(new StringReader(privateKeyString))) {
              privateKeyPem = reader.readPemObject();
          }

          KeyFactory keyFactory = KeyFactory.getInstance("RSA");
          PrivateKey privateKey = keyFactory.generatePrivate(new PKCS8EncodedKeySpec(privateKeyPem.getContent()));

          Instant now = Instant.now();

          // JWT generation.
          String encodedToken = Jwts.builder()
                  .setHeaderParam("kid", keyId)
                  .setIssuer(serviceAccountId)
                  .setAudience("https://iam.{{ api-host }}/iam/v1/tokens")
                  .setIssuedAt(Date.from(now))
                  .setExpiration(Date.from(now.plusSeconds(3600)))
                  .signWith(privateKey, SignatureAlgorithm.PS256)
                  .compact();
          System.out.println(encodedToken);
      }
  }
  ```

- C# {#csharp}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [jose-jwt](https://www.nuget.org/packages/jose-jwt/):
  - Verified for jose-jwt 5.0.0.

  **Net Framework / Net Core**:

    Verified for Net Framework 4.8.1 and Net Core 3.1.

    ```c#
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Security.Cryptography;
    using Jose;
    using Org.BouncyCastle.Crypto.Parameters;
    using Org.BouncyCastle.OpenSsl;
    using Org.BouncyCastle.Security;
    
    namespace ConsoleApp
    {
        class Program
        {
            static void Main(string[] args)
            {
                var serviceAccountId = "<service_account_ID>";
                var keyId = "<public_key_ID>";
                var now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
    
                var headers = new Dictionary<string, object>()
                {
                    { "kid", keyId }
                };
    
                var payload = new Dictionary<string, object>()
                {
                    { "aud", "https://iam.{{ api-host }}/iam/v1/tokens" },
                    { "iss", serviceAccountId },
                    { "iat", now },
                    { "exp", now + 3600 }
                };
    
                RsaPrivateCrtKeyParameters privateKeyParams;
                using (var pemStream = File.OpenText("<private_key_file>"))
                {
                    privateKeyParams = new PemReader(pemStream).ReadObject() as RsaPrivateCrtKeyParameters;
                }
    
                using (var rsa = RSA.Create())
                {
                    rsa.ImportParameters(DotNetUtilities.ToRSAParameters(privateKeyParams));
                    string encodedToken = Jose.JWT.Encode(payload, rsa, JwsAlgorithm.PS256, headers);
                }
            }
        }
    }
    ```

  **.NET 5.0+**:
    
    Verified for NET 5.0, NET 6.0, NET 7.0, and NET 8.0.

    ```c#
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Security.Cryptography;
    using Jose;

    namespace ConsoleApp
    {
        class Program
        {
            static void Main(string[] args)
            {
                var serviceAccountId = "<service_account_ID>";
                var keyId = "<public_key_ID>";
                var now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();

                var headers = new Dictionary<string, object>()
                {
                    { "kid", keyId }
                };

                var payload = new Dictionary<string, object>()
                {
                    { "aud", "https://iam.{{ api-host }}/iam/v1/tokens" },
                    { "iss", serviceAccountId },
                    { "iat", now },
                    { "exp", now + 3600 }
                };

                using (var rsa = RSA.Create())
                {
                    rsa.ImportFromPem(File.ReadAllText("<private_key_file>").ToCharArray());
                    string encodedToken = Jose.JWT.Encode(payload, rsa, JwsAlgorithm.PS256, headers);
                }
            }
        }
    }
    ```

- Go {#go}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [golang-jwt](https://github.com/golang-jwt/jwt).
  - Tested for Go 1.23.1 and golang-jwt v5.
  - Private key is read from the JSON file obtained when creating the authorized key.

  Install the required packages:

  ```
  install jwt v5
  go get -u github.com/golang-jwt/jwt/v5
  ```

    
  ```go
  package main  
  
  import (
  	"crypto/rsa"
  	"encoding/json"
  	"log"
  	"os"
  	"time"
  
  	"github.com/golang-jwt/jwt/v5"
  )
  
  func main() {
    // Getting a token
  	token := signedToken()
    // Saving the token to a file
  	err := os.WriteFile("jwt_token.txt", []byte(token), 0644)
  	if err != nil {
  	  log.Fatal(err)
  	}
  	// Printing the token to console
  	fmt.Println("Here is token:")
  	fmt.Println(token)
  }
  
  const (
    keyID            = "<public_key_ID>"
    serviceAccountID = "<service_account_ID>"
    keyFile          = "<JSON_file_with_keys>"
  )
  
  // JWT generation.
  func signedToken() string {
  	claims := jwt.RegisteredClaims{
  	 	Issuer:    serviceAccountID,
  	 	ExpiresAt: jwt.NewNumericDate(time.Now().UTC().Add(1 * time.Hour)),
  	 	IssuedAt:  jwt.NewNumericDate(time.Now().UTC()),
  	 	NotBefore: jwt.NewNumericDate(time.Now().UTC()),
  	 	Audience:  []string{"https://iam.{{ api-host }}/iam/v1/tokens"},
  	}
  	token := jwt.NewWithClaims(jwt.SigningMethodPS256, claims)
  	token.Header["kid"] = keyID
  
  	privateKey := loadPrivateKey()
  	signed, err := token.SignedString(privateKey)
  	if err != nil {
  		panic(err)
  	}

  	return signed
  }
  
  func loadPrivateKey() *rsa.PrivateKey {
    keyData := readPrivateKey()

    rsaPrivateKey, err := jwt.ParseRSAPrivateKeyFromPEM([]byte(keyData.PrivateKey))
    if err != nil {
      panic(err)
    }
    return rsaPrivateKey
	}

  func readPrivateKey() *iamkey.Key {
    data, err := os.ReadFile(keyFile)
    if err != nil {
      panic(err)
    }

    var keyData *iamkey.Key
    if err := json.Unmarshal(data, &keyData); err != nil {
      panic(err)
    }

    return keyData
  }
  ```


- Node.js {#node}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [node-jose](https://github.com/cisco/node-jose):
  - Verified for Node.js v20.12.1 and node-jose 2.2.0.
  - Required data is read from the JSON file obtained when creating the authorized key.

  
  ```js
  const serviceAccountJson = require('<JSON_file_with_keys>')
  const jose = require('node-jose');

  const {
      id: accessKeyId,
      service_account_id: serviceAccountId,
      private_key: privateKey
  } = serviceAccountJson

  const createJWT = () =>
  {
      const now = Math.floor(new Date().getTime() / 1000)
      const payload = {
          iss: serviceAccountId,
          iat: now,
          exp: now + 3600,
          aud: "https://iam.api.cloud.yandex.net/iam/v1/tokens"
      
      return jose.JWK.asKey(privateKey, 'pem', { kid: accessKeyId, alg: 'PS256' })
          .then(function (result)
          {
              return jose.JWS.createSign({ format: 'compact' }, result)
                  .update(JSON.stringify(payload))
                  .final();
          });
  }
  ```

  

- PHP {#php}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [PHP JWT Framework](https://github.com/web-token/jwt-framework):
  - Verified for PHP v8.3.4 and web-token/jwt-framework v3.3.5.
  - Verified for PHP v7.4.33 and web-token/jwt-framework v2.2.11.
  - Required data is read from the JSON file obtained when creating the authorized key.


  ```php
  require 'vendor/autoload.php';
  
  use Jose\Component\Core\AlgorithmManager;
  use Jose\Component\KeyManagement\JWKFactory;
  use Jose\Component\Signature\Algorithm\PS256;
  use Jose\Component\Signature\JWSBuilder;
  use Jose\Component\Signature\Serializer\CompactSerializer;
  
  // Reading data from a file
  $keyData = json_decode(file_get_contents("<JSON_file_with_keys>"), true);
  $privateKeyPem = $keyData['private_key'];
  $keyId = $keyData['id'];
  $serviceAccountId = $keyData['service_account_id'];
  
  // You need to delete header/metadata from the private key
  if (strpos($privateKeyPem, "PLEASE DO NOT REMOVE THIS LINE!") === 0) {
      $privateKeyPem = substr($privateKeyPem, strpos($privateKeyPem, "\n") + 1);
  }
  
  $jwk = JWKFactory::createFromKey(
      $privateKeyPem, 
      null, 
      [
          'alg' => 'PS256',
          'use' => 'sig',
          'kid' => $keyId,
      ]
  );
  
  $algorithmManager = new AlgorithmManager([new PS256()]);
  $jwsBuilder = new JWSBuilder($algorithmManager);
  
  $payload = json_encode([
      'iss' => $serviceAccountId,
      'aud' => "https://iam.{{ api-host }}/iam/v1/tokens",
      'iat' => time(),
      'nbf' => time(),
      'exp' => time() + 3600,
  ]);
  
  $jws = $jwsBuilder
      ->create()
      ->withPayload($payload)
      ->addSignature($jwk, ['alg' => 'PS256', 'typ'=>'JWT', 'kid' => $keyId])
      ->build();
  
  
  $serializer = new CompactSerializer();
  $token = $serializer->serialize($jws, 0); 
  
  // Saving the token to a file
  file_put_contents('jwt_token.txt', $token);
  // Printing the token to console
  echo "JWT Token: " . $token . PHP_EOL;
  ```

- C++ {#cpp}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [jwt-cpp](https://github.com/Thalhammer/jwt-cpp):
  - Verified for C++ 14 and jwt-cpp 0.7.0.
  - Required data is read from the JSON file obtained when creating the authorized key.

  ```cpp
  #include <chrono>
  #include <fstream>
  #include <iterator>
  
  #include "jwt-cpp/jwt.h"
  
  int main()
  {
      std::ifstream key_file("<JSON_file_with_keys>");
      std::string content((std::istreambuf_iterator<char>(key_file)),
          (std::istreambuf_iterator<char>()));
  
      picojson::value v;
      std::string err = picojson::parse(v, content);
      auto privateKey = v.get("private_key").to_str();
      auto serviceAccountId = v.get("service_account_id").to_str();
      auto keyId = v.get("id").to_str();
  
      auto now = std::chrono::system_clock::now();
      auto expires_at = now + std::chrono::hours(1);
      picojson::array audience_array;
      audience_array.push_back(picojson::value("https://iam.{{ api-host }}/iam/v1/tokens"));
      auto algorithm = jwt::algorithm::ps256(
          "",
          privateKey);
  
      // JWT generation.
      auto encoded_token = jwt::create()
          .set_key_id(keyId)
          .set_issuer(serviceAccountId)
          .set_audience(audience_array)
          .set_issued_at(now)
          .set_expires_at(expires_at)
          .sign(algorithm);
  
      std::cout << encoded_token;
  }
  ```

- Ruby {#ruby}

  {% include [jwt-external-libs-notice](../../../_includes/iam/jwt-external-libs-notice.md) %}

  Example of creating a JWT using [ruby-jwt](https://github.com/jwt/ruby-jwt):
  - Verified for Ruby 3.2.3 and jwt 2.8.1.
  - Required data is read from the JSON file obtained when creating the authorized key.

  Install the jwt package:

  ```
  gem install jwt
  ```

  ```ruby
  
  require 'jwt'
  require 'json'
  require 'time'
  
  KEY_FILE = '<JSON_file_with_keys>'
  KEY_DATA = JSON.parse(File.read(KEY_FILE))
  KEY_ID = KEY_DATA['id']
  SERVICE_ACCOUNT_ID = KEY_DATA['service_account_id']
  
  def load_private_key
    OpenSSL::PKey::RSA.new(KEY_DATA['private_key'])
  rescue IOError, JSON::ParserError, OpenSSL::PKey::RSAError => e
    raise "Failed to load or parse private key: #{e.message}"
  end
  
  def signed_token
    payload = {
      iss: SERVICE_ACCOUNT_ID,
      exp: Time.now.to_i + 3600,
      iat: Time.now.to_i,
      nbf: Time.now.to_i,
      aud: "https://iam.{{ api-host }}/iam/v1/tokens"
    }
  
    header = {
      kid: KEY_ID
    }
  
    private_key = load_private_key
  
    JWT.encode(payload, private_key, 'PS256', header)
  end
  
  # Main execution
  begin
    token = signed_token
    File.write('jwt_token.txt', token)
    # Or, alternatively, print the token to the console
    # puts "Here is the token:"
    # puts token
  rescue => e
    puts "An error occurred: #{e.message}"
  end
  ```

{% endlist %}


### 2. Exchange the JWT for an IAM token {#get-iam-token}

When exchanging the JWT for an IAM token, make sure the following conditions are met:
* The service account and key specified in the JWT exist (they have not been deleted).
* The key belongs to the service account.
* The signature is valid.

{% list tabs group=programming_language %}

- API {#api}

  To get an IAM token, use the [create](../../api-ref/IamToken/create.md) REST API method for the [IamToken](../../api-ref/IamToken/index.md) resource or the [IamTokenService/CreateForServiceAccount](../../api-ref/grpc/IamToken/createForServiceAccount.md) gRPC API call.

  Request example with cURL for the `create` REST API method:

  ```curl
  curl \
      --request POST \
      --header 'Content-Type: application/json' \
      --data '{"jwt": "<JWT>"}' \
      https://iam.{{ api-host }}/iam/v1/tokens
  ```

  Where `<JWT_token>` is the JWT you got in the previous step.

- Python {#python}

  To exchange your JWT for an IAM token, you can use the [YC-SDK](../../../overview/sdk/overview.md) for Python.
  1. Install the {{ yandex-cloud }} SDK for Python: 

      ```bash
      pip install yandexcloud
      ```

  1. Exchange your JWT for an IAM token:

      ```go
      import yandexcloud

      from yandex.cloud.iam.v1.iam_token_service_pb2 import (CreateIamTokenRequest)
      from yandex.cloud.iam.v1.iam_token_service_pb2_grpc import IamTokenServiceStub

      key_path = '<JSON_file_with_keys>'

      # Reading a private key from a JSON file
      with open(key_path, 'r') as f:
        obj = f.read() 
        obj = json.loads(obj)
        private_key = obj['private_key']
        key_id = obj['id']
        service_account_id = obj['service_account_id']

      sa_key = {
          "id": key_id,
          "service_account_id": service_account_id,
          "private_key": private_key
      }

      def create_iam_token():
        jwt = create_jwt()
        
        sdk = yandexcloud.SDK(service_account_key=sa_key)
        iam_service = sdk.client(IamTokenServiceStub)
        iam_token = iam_service.Create(
            CreateIamTokenRequest(jwt=jwt)
        )
        
        print("Your iam token:")
        print(iam_token.iam_token)

        return iam_token.iam_token
      ```

- Go {#go}

  To exchange your JWT for an IAM token, you can use the [YC-SDK](../../../overview/sdk/overview.md) for Go.
  1. Install the {{ yandex-cloud }} SDK for Go: 

      ```bash
      go get github.com/yandex-cloud/go-sdk
      ```

  1. Exchange your JWT for an IAM token:

      ```go
      import (
        "context"
        
        "github.com/yandex-cloud/go-genproto/yandex/cloud/iam/v1"
        ycsdk "github.com/yandex-cloud/go-sdk"
      )

      func getIAMToken() string {
        authKey := readPrivateKey()
        ctx := context.Background()

        credentials, err := ycsdk.ServiceAccountKey(authKey)
        if err != nil {
          panic(err)
        }

        sdk, err := ycsdk.Build(ctx, ycsdk.Config{
          Credentials: credentials,
        })
        if err != nil {
          panic(err)
        }

        iamRequest := &iam.CreateIamTokenRequest{
          Identity: &iam.CreateIamTokenRequest_Jwt{Jwt: signedToken()},
        }

        newKey,err := sdk.IAM().IamToken().Create(ctx,iamRequest)
        if err != nil {
          panic(err)
        }

        fmt.Println("Your iam token:")
        fmt.Println(newKey.IamToken)
        return newKey.IamToken
      }
      ```

- Node.js {#node}

  To exchange your JWT for an IAM token, you can use the [YC-SDK](../../../overview/sdk/overview.md) for Node.js.
  1. Install the {{ yandex-cloud }} SDK for Node.js: 

      ```bash
      npm install @yandex-cloud/nodejs-sdk
      ```

  1. Exchange your JWT for an IAM token:

      ```js
      const serviceAccountJson = require('<JSON_file_with_keys>')
      const {
          serviceClients, Session, cloudApi, waitForOperation, decodeMessage,
      } = require('@yandex-cloud/nodejs-sdk');

      const {
          id: accessKeyId,
          service_account_id: serviceAccountId,
          private_key: privateKey
      } = serviceAccountJson

      const {
          iam: {
              iam_token_service: {
                  CreateIamTokenRequest,
              }
          }
      } = cloudApi;

      async function createIamToken()
      {
          const session = new Session({
              serviceAccountJson: {
                  accessKeyId,
                  serviceAccountId,
                  privateKey,
              }
          })
          const tokenClient = session.client(serviceClients.IamTokenServiceClient)
          const jwt = await createJWT()
          const tokenRequest = CreateIamTokenRequest.fromPartial({ jwt })
          const { iamToken } = await tokenClient.create(tokenRequest)

          console.log("Your iam token:")
          console.log(iamToken)

          return iamToken
      }

      createIamToken()
      ```

{% endlist %}

{% include [iam-token-usage](../../../_includes/iam-token-usage.md) %}


#### What's next {#what-is-next}

* [{#T}](../sa/set-access-bindings.md)
* [{#T}](../sa/assign-role-for-sa.md)
* [{#T}](./revoke-iam-token.md)
