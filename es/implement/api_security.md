# Seguridad API REST

El API REST Existente debe implementar algún mecanismo de seguridad

  * Basic Auth
  * API Key
  * OAuth
  * Personal API Token
  * etc.

## Personal API Token

El mecanismo de Personal API Token es uno muy utilizado, sin embargo, no es el más seguro. Los API Tokens funcionan como passwords, la ventaja es que pueden ser revocados.

Consideraciones:

1. Proveer un mecanismo para **generar** Personal API Tokens al usuario
2. Proveer un mecanismo para **revocar** Personal API Tokens
3. El API Token generalmente viaja en los headers en cada llamada al API REST

Por favor visitar https://jwt.io/introduction/, que provee un estándar (Json web tokens) de autenticación.
