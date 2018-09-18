# Credit Card Terminal HTTP API

Controll a credit card terminal through this Rest API.
The Rest API is installed as a Windows Service.
The credit card terminal can be connected using TCP/IP or RS232.

The API implements the ZVT protocoll which is commonly used in the "DACH" region (Germany, Austria, Switzerland). It standardises the communication between any* credit card terminal, machine terminal or handeld.

*please refer to the interface specifications from the terminal you would like to connect to.
Your Payment Service Provider (PSP) may provide additional information.

Tested with [CCV - OPP C60](https://www.ccv.eu/ch-de/home/produktuebersicht/terminals/automatenmodule/)

## Basic Protocol Structure

The protocol data packet is a JSON formatted, UTF-8 encoded string.
If a command is sent, the answer regarding JSON format and parameter validation is an immediate responded by the server.
The variable "Proceed" indicates if the command is executed by the server or not. If the command gets executed, the client needs to track the HTTP calls generated by the server and parse the JSON messages in the POST body.

The work flow is as follows:

- Client sends command.
- Server validates the message and sends an immediate response.
- If (Proceed == true) the server stars executing and performs the ZVT terminal communication
- The client has to track the intermediate (Responses on device level) messages to display infos
  The intermediate messages are sent to the `CallbackUrl`.
- There can be multiple intermediate messages where the field "IntermediateState" is set to TRUE
- The procedure ends with one message where the field "IntermediateState" is set to FALSE (The end message)

### Request

```json
{
    "Command":"<given command>",
    "CallbackUrl":"<The url to send intermediate states to>",
    "Parameter":"[optional]"
}
```

### Response

```json
{
    "HttpStatus": "<the http state of the response>",
    "Message": "<an optional message>",
    "Proceed": "<bool - defining if the command has been acceptet and is executed>",
    "SessionId": "<sessionid>"
}
```

### Intermediate messages

> sent to `CallbackUrl`

```json
{
    "Command":"<sent command>",
    "Success":"<bool - is the response succesful>",
    "Value":"<optional base64 encoded message>",
    "IsIntermediateState":"<bool - defines if the response is an intermediate state>",
    "ReceiptNumber":"int - <The receipt number of the transaction -1 if no receipt number was given>",
    "ReceiptText":"<an array of base64 encoded lines defining the receipt text>"
}
```

## Commands

The commands are case sensitive strings in exact spelling like the following headers:

### IsIdle

The command asks the server if it is ready for execution.
If "Success" is TRUE the server has no pending operations.

### DiagnosisSimple

Server starts and Diagnosis and end only with one end intermediate state.

### Authorisation

The command starts a payment procedure.
The field "Parameter" has to be set as string with the amount of payment IN CENTS.

### Reversal

The server starts the reversal of the previous payment. Only the previous payment can be reversed.
The user has to insert the card and enter his pincode again.

### EndOfDay

Server starts an end of day procedure. This is usually also done automatically by the terminal over night.

### CancelTransaction

The server cancels a pending transaction if it is possible (ex: before entering the pin code).
Depending on the state of the procedure to cancel one or more intermediate states are possible.

### GetAbortedSessions

Transactions can be aborted either by the user at the terminal or by software with the interface. If intermediate states cannot get sent back to the php client
the server aborts the transaction as well.
The server stores every aborted transaction, unfinished or invalid session in a local database. 
The command 'GetAbortedSessions' sends the content of this database in an intermediate state to the client.

### RemoveAbortedSession

If the aborted session was already processed by the client it can be removed from the server by using the 'RemoveAbortedSession' command and 
providing the ID o the session in the 'Parameter' field. The result is sent back in the Success-Field of the intermediate state.

## Licensing

Copyright Key & Card AG
[Product license available here](https://check24-7.in/kontakt/offerte)
