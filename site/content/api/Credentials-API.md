+++
title = "Credentials API"
weight = 435
+++

The *Credentials API* is used by *Protocol Adapters* to retrieve credentials used to authenticate *Devices* connecting to the adapter. In particular, the API supports the storage, look up and deletion of *shared secrets* which are often used by IoT devices by means of *username/password* based authentication schemes.
<!--more-->

Credentials are of a certain *type* which indicates which authentication mechanism the credentials can be used with. Each set of credentials also contains an *authentication identity* which is the identity claimed by the device during authentication. This authentication identity is usually different from the *logical* ID the device has been registered with using Hono's [Device Registration API]({{< ref "api/Device-Registration-API.md" >}}). Multiple sets of credentials (including arbitrary *authentication identities*) can be registered for each *logical* device ID.

Note, however, that in real world applications the device credentials will probably be managed by a dedicated identity management system. In such a case an implementation of Hono's Credentials API can be created which maps the AMQP 1.0 based API calls to the identity management system's (remote) API.

The Credentials API is defined by means of AMQP 1.0 message exchanges, i.e. a client needs to connect to Hono using an AMQP 1.0 client in order to invoke operations of the API as described in the following sections.

# Preconditions

The preconditions for invoking any of the Credential API's operations are as follows:

1. Client has established an AMQP connection to Hono.
2. Client has established an AMQP link in role *sender* with Hono using target address `credentials/${tenant_id}`. This link is used by the client to send commands to Hono.
3. Client has established an AMQP link in role *receiver* with Hono using source address `credentials/${tenant_id}/${reply-to}` where *reply-to* may be any arbitrary string chosen by the client. This link is used by the client to receive responses to the requests it has sent to Hono. This link's source address is also referred to as the *reply-to* address for the request messages.

# Operations

The operations described in the following sections can be used by clients to manage credentials for authenticating devices connected to protocol adapters.

{{% note %}}
All operations are scoped to the *tenant* specified by the `${tenant_id}` during link establishment. It is therefore not possible to e.g. add credentials for devices of *TENANT_B* if the link has been established for target address `credentials/TENANT_A`.
{{% /note %}}

## Add Credentials

Clients use this command to initially *add* credentials for a device that has already been registered with Hono. The credentials to be added may be of arbitrary type. However, [Standard Credential Types]({{< relref "#standard-credential-types" >}}) contains an overview of some common types that are used by Hono's protocol adapters and which may be useful to others as well.

**Message Flow**

*TODO* add sequence diagram

**Request Message Format**

The following table provides an overview of the properties a client needs to set on an *add credentials* message in addition to the [Standard Request Properties]({{< relref "#standard-request-properties" >}}).

| Name             | Mandatory | Location                 | Type            | Description                         |
| :--------------- | :-------: | :----------------------- | :-------------- | :---------------------------------- |
| *subject*        | yes       | *properties*             | UTF-8 *string*  | MUST contain the value `add`.      |

The request message MUST include payload as defined in [Request Payload]({{< relref "#request-payload" >}}).

**Response Message Format**

A response to an *add credentials* request contains the [Standard Response Properties]({{< relref "#standard-response-properties" >}}).

The response message's body is empty. 

The response message's *status* property may contain the following codes:

| Code  | Description |
| :---- | :---------- |
| *201* | Created, the credentials have been added successfully. |
| *409* | Conflict, there already exist credentials with the `type` and `auth-id` for the `device-id` from the payload. |
| *412* | Precondition Failed, there is no device registered with the given *device-id* within the tenant. |

For status codes indicating an error (codes in the `400 - 499` range) the message body MAY contain a detailed description of the error that occurred.

## Get Credentials

Protocol adapters use this command to *look up* credentials of a particular type for a device identity.

**Message Flow**

*TODO* add sequence diagram

**Request Message Format**

The following table provides an overview of the properties a client needs to set on an *get credentials* message in addition to the [Standard Request Properties]({{< relref "#standard-request-properties" >}}).

| Name             | Mandatory | Location                 | Type            | Description                   |
| :--------------- | :-------: | :----------------------- | :-------------- | :---------------------------- |
| *subject*        | yes       | *properties*             | UTF-8 *string*  | MUST contain the value `get`. |

The body of the message MUST consist of a single *AMQP Value* section containing a UTF-8 encoded string representation of a single JSON object having the following members:

| Name             | Mandatory | Type       | Description |
| :--------------- | :-------: | :--------- | :---------- |
| *type*           | *yes*     | *string*   | The type of credentials to look up. Potential values include (but are not limited to) `psk`, `RawPublicKey`, `hashed-password` etc. |
| *auth-id*        | *yes*     | *string*   | The authentication identifier to look up credentials for. |

The following request payload may be used to look up the hashed password for user `billie`:

~~~json
{
  "type": "hashed-password",
  "auth-id": "billie"
}
~~~

**Response Message Format**

A response to a *get credentials* request contains the [Standard Response Properties]({{< relref "#standard-response-properties" >}}).

The response message includes payload as defined in [Response Payload]({{< relref "#response-payload" >}}).

The response message's *status* property may contain the following codes:

| Code  | Description |
| :---- | :---------- |
| *200* | OK, the payload contains the credentials for the authentication identifier. |
| *404* | Not Found, there are no credentials registered matching the critera. |

For status codes indicating an error (codes in the `400 - 499` range) the message body MAY contain a detailed description of the error that occurred.

## Update Credentials

Clients use this command to *update* existing credentials registered for a device. All of the information that has been previously registered for the device gets *replaced* with the information contained in the request message.

**Message Flow**

*TODO* add sequence diagram

**Request Message Format**

The following table provides an overview of the properties a client needs to set on an *update credentials* message in addition to the [Standard Request Properties]({{< relref "#standard-request-properties" >}}).

| Name             | Mandatory | Location                 | Type           | Description |
| :--------------- | :-------: | :----------------------- | :------------- | :---------- |
| *subject*        | yes       | *properties*             | UTF-8 *string* | MUST contain the value `update`. |

The request message MUST contain payload as defined in [Request Payload]({{< relref "#request-payload" >}}).

**Response Message Format**

A response to an *update credentials* request contains the [Standard Response Properties]({{< relref "#standard-response-properties" >}}).

The response message's body is empty. 

The response message's *status* property may contain the following codes:

| Code  | Description |
| :---- | :---------- |
| *204* | No Content, the credentials have been updated successfully. |
| *404* | Not Found, there are no credentials registered matching the critera from the payload. |

For status codes indicating an error (codes in the `400 - 499` range) the message body MAY contain a detailed description of the error that occurred.

## Remove Credentials

Clients use this command to *remove* credentials registered for a device. Once the credentials are removed, the device may no longer be able to authenticate with protocol adapters using the authentication mechanism that the removed credentials corresponded to.

**Message Flow**

*TODO* add sequence diagram

**Request Message Format**

The following table provides an overview of the properties a client needs to set on a *remove credentials* message in addition to the [Standard Request Properties]({{< relref "#standard-request-properties" >}}).

| Name             | Mandatory | Location                 | Type           | Description |
| :--------------- | :-------: | :----------------------- | :------------- | :---------- |
| *subject*        | yes       | *properties*             | UTF-8 *string* | MUST contain the value `remove`. |

The body of the message MUST consist of a single *AMQP Value* section containing a UTF-8 encoded string representation of a single JSON object as follows:

~~~json
{
  "device-id": "${device_id}",
  "type": "${credential_type}",
  "auth-id": "${authentication_identifier}"
}
~~~

The `device-id` property MUST contain the ID of the device from which the credentials should be removed.
The `type` property indicates the type of credentials to remove. If set to `*` then all credentials of the device are removed (`auth-id` is ignored), otherwise only the credentials matching the `type` and `auth-id` are removed.
The `auth-id` property is optional and if omitted, then all credentials of the specified `type` are removed.

**Response Message Format**

A response to a *remove credentials* request contains the [Standard Response Properties]({{< relref "#standard-response-properties" >}}).

The response message's body is empty.

The response message's *status* property may contain the following codes:

| Code  | Description |
| :---- | :---------- |
| *204* | No Content, the credentials have been removed successfully. |
| *404* | Not Found, there are no credentials registered matching the critera given in the request payload. |

For status codes indicating an error (codes in the `400 - 499` range) the message body MAY contain a detailed description of the error that occurred.

# Standard Message Properties

Due to the nature of the request/response message pattern of the operations of the Credentials API, there are some standard properties shared by all of the request and response messages exchanged as part of the operations.

## Standard Request Properties

The following table provides an overview of the properties shared by all request messages regardless of the particular operation being invoked.

| Name             | Mandatory | Location                 | Type           | Description |
| :--------------- | :-------: | :----------------------- | :------------- | :---------- |
| *action*         | yes       | *application-properties* | UTF-8 *string* | MUST be set to the value defined by the particular operation being invoked. |
| *correlation-id* | no        | *properties*             | *message-id*   | MAY contain an ID used to correlate a response message to the original request. If set, it is used as the *correlation-id* property in the response, otherwise the value of the *message-id* property is used. |
| *message-id*     | yes       | *properties*             | UTF-8 *string* | MUST contain an identifier that uniquely identifies the message at the sender side. |
| *reply-to*       | yes       | *properties*             | UTF-8 *string*  | MUST contain the source address that the client wants to received response messages from. This address MUST be the same as the source address used for establishing the client's receive link (see [Preconditions]({{< relref "#preconditions" >}}). |

## Standard Response Properties

The following table provides an overview of the properties shared by all response messages regardless of the particular operation being invoked.

| Name             | Mandatory | Location                 | Type            | Description |
| :--------------- | :-------: | :----------------------- | :-------------- | :---------- |
| *correlation-id* | yes       | *properties*             | *message-id*    | Contains the *message-id* (or the *correlation-id*, if specified) of the request message that this message is the response to. |
| *device_id*      | yes       | *application-properties* | UTF-8 *string*  | Contains the ID of the device. |
| *tenant_id*      | yes       | *application-properties* | UTF-8 *string*  | Contains the ID of the tenant to which the device belongs. |
| *status*         | yes       | *application-properties* | *int*           | Contains the status code indicating the outcome of the operation. Concrete values and their semantics are defined for each particular operation. |

# Delivery States

Hono uses the following AMQP message delivery states when receiving request messages from clients:

| Delivery State | Description |
| :------------- | :---------- |
| *ACCEPTED*     | Indicates that Hono has successfully received and accepted the request for processing. |
| *REJECTED*     | Indicates that Hono has received the request but was not able to process it. The *error* field contains information regarding the reason why. Clients should not try to re-send the request using the same message properties in this case. |

# Payload Format

Most of the operations of the Credentials API allow or require the inclusion of payload data as part of the
request or response messages of the operation. Such payload is carried in the body of the corresponding AMQP 
messages as part of a single *AMQP Value* section.

## Request Payload

The payload included in *request* messages consists of a UTF-8 encoded string representation of a single JSON object. It is an error to include payload that is not of this type.

The table below provides an overview of the standard members defined for the JSON object:

| Name             | Mandatory | Type       | Default Value | Description |
| :--------------- | :-------: | :--------- | :------------ | :---------- |
| *device-id*      | *yes*     | *string*   |               | The ID of the device to which the credentials belong. |
| *type*           | *yes*     | *string*   |               | The credential type name. The value may be arbitrarily chosen by clients but SHOULD reflect the particular type of authentication mechanism the credentials are to be used with. Possible values include (but are not limited to) `psk`, `RawPublicKey`, `hashed-password` etc. |
| *auth-id*        | *yes*     | *string*   |               | The identity that the device should be authenticated as. |
| *enabled*        | *no*      | *boolean*  | *true*        | Indicates whether the credentials can be used to authenticate devices. **NB** It is up to the discretion of the protocol adapter to make use of this information. |
| *not-before*     | *no*      | *string*   | *null*        | The point in time from which on the credentials can be used to authenticate devices. If not *null*, the value MUST be an [ISO 8601 compliant *combined date and time representation*](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). **NB** It is up to the discretion of the protocol adapter to make use of this information. |
| *not-after*      | *no*      | *string*   | *null*        | The point in time until which the credentials can be used to authenticate devices. If not *null*, the value MUST be an [ISO 8601 compliant *combined date and time representation*](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). **NB** It is up to the discretion of the protocol adapter to make use of this information. |

For each set of credentials the combination of `auth-id` and `type` MUST be unique within a tenant.

The JSON object MAY contain additional members of arbitrary other names which MUST be of a scalar type only.

Below is an example for a payload of a request for adding [a hashed password]({{< ref "#hashed-password" >}}) for username `billie` using SHA512 as the hashing function with a 4 byte salt (Base64 encoding of `0x32AEF017`) to device `4711`:

~~~json
{
  "device-id": "4711",
  "type": "hashed-password",
  "auth-id": "billie",
  "pwd-hash": "AQIDBAUGBwg=",
  "salt": "Mq7wFw==",
  "hash-function": "sha512"
}
~~~

## Response Payload

The payload included in *response* messages consists of a UTF-8 encoded string representation of a single JSON object.
The object contains the credential properties registered for the device.

Below is an example of the resulting payload for a *get* request for `hashed-password`-type credentials for username `billie`:

~~~json
{
  "device-id": "4711",
  "type": "hashed-password",
  "auth-id": "billie",
  "pwd-hash": "AQIDBAUGBwg=",
  "salt": "Mq7wFw==",
  "hash-function": "sha512"
}
~~~

# Standard Credential Types

The following sections define some standard credential types and their properties. Applications are encouraged to make use of these types. However, the types are not enforced anywhere in Hono and clients may of course add application specific properties to the credential types.

## Common Properties

All credential types used with Hono MUST contain a `device-id`, `type` and `auth-id` property as defined in [Request Payload]({{< relref "#request-payload" >}}).

## Hashed Password

A credential type for storing a (hashed) password for a user.

Example:

~~~json
{
  "device-id": "4711",
  "type": "hashed-password",
  "auth-id": "billie",
  "pwd-hash": "AQIDBAUGBwg=",
  "salt": "Mq7wFw==",
  "hash-function": "sha512"
}
~~~

| Name             | Mandatory | Type       | Default   | Description |
| :--------------- | :-------: | :--------- | :-------- | :---------- |
| *device-id*      | *yes*     | *string*   |           | The ID of the device to which the credentials belong. |
| *type*           | *yes*     | *string*   |           | The credential type name, always `hashed-password`. |
| *auth-id*        | *yes*     | *string*   |           | The *username* |
| *pwd-hash*       | *yes*     | *string*   |           | The Base64 encoded bytes representing the hashed password. |
| *salt*           | *no*      | *string*   |           | The Base64 encoded bytes used as *salt* for the password hash. If not set then the password hash has been created without salt. |
| *hash-function*  | *no*      | *string*   | `sha256`  | The name of the hash function used to create the password hash. Examples include `sha256`, `sha512` etc. |

## Pre-Shared Key

A credential type for storing a *Pre-shared Key* as used in TLS handshakes.

Example:

~~~json
{
  "device-id": "4711",
  "type": "psk",
  "auth-id": "little-sensor",
  "key": "AQIDBAUGBwg="
}
~~~

| Name             | Mandatory | Type       | Description |
| :--------------- | :-------: | :--------- | :---------- |
| *device-id*      | *yes*     | *string*   | The ID of the device to which the credentials belong. |
| *type*           | *yes*     | *string*   | The credential type name, always `psk`. |
| *auth-id*        | *yes*     | *string*   | The PSK identifier. |
| *key*            | *yes*     | *string*   | The Base64 encoded bytes representing the shared (secret) key. |
