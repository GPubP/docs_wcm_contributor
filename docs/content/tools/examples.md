# Voorbeelden

!> **Todo** Hier komt een voorbeeld/start swagger...


>[!note|icon:fas fa-download|label:Download]
> je kan hier een basis file downloaded, zeg maar een bootstrap [starter.oas3.json](/downloads)


``` yaml
openapi: 3.0.5
info:
  version: '1.0.0'
  title: 'Sales Invoice - Part I'
  description: 'This is an example API used for training purposes of Digipolis colleagues and their partners. '
servers:
  - url: https://api.example.com/v1
    description: Production server (uses live data)
  - url: https://sandbox-api.example.com:8443/v1
    description: Sandbox server (uses test data)
paths:
  /invoices:
    get:
      description: Get all invoices from this wonderful API ;). Make sure you provide the proper header to work in the right tenant. Use the `tenant-key` header.
      summary: Get all invoices
      responses:
        '200':
          description: OK
    post:
      description: Add a new invoice to the system.
      summary: Add a new invoice.
      responses:
        '200':
          description: OK
  /invoices/{id}:
    get:
      description: Retrieve an invoice for the given invoice number (the `id` in the path).
      summary: Retrieve the data for one specific invoice
      parameters:
        - in: path
          name: id
          schema:
            type: string
          required: true
          description: The invoice number.
          example: 'INV0001'
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema: 
                $ref: '#/components/schemas/invoice'
        '404':
          description: Not Found
components:
  schemas:
    invoice:
      type: object
      properties:
        number: 
          type: string
          description: A unique sequential invoice number, prefixed with 'INV'
          example: INV10034
        date:
          type: string
          format: date
          description: The date when this invoice was issued.
          example: 28-04-2020
```