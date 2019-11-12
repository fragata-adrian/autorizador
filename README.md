# Autorizador

## Operaciones
El programa maneja dos tipos de operaciones. Ambos operaciones deben exponerse como servicios RESTful independientes que reciban y retornen documentos json.

1. Creación de cuenta
2. Autorización de transacción

### Account creation

#### Entrada
Crea la cuenta con setLimit y activeCard disponibles. La aplicación debe soportar multiples cuentas.
Si una cuenta ya exíste, debe informarlo en la salida.

#### Salida
El estado actual de la cuenta creada + cualquier violación de lógica de negocios.
Violaciones de lógica de negocios
 
#### Ejemplos de entrada
```json
    { "account": { "id": "3bjk2h39", "activeCard": true, "availableLimit": 100 } }
    { "account": { "id": "3lkj4ddd", "activeCard": false, "availableLimit": 4000 } }

```

#### Ejemplos de salida

```json
    { "account": { "id": "3bjk2h39", "activeCard": true, "availableLimit": 100 }, "violations": [] }

    { "account": { "id": "3lkj4ddd",  "activeCard": true, "availableLimit": 100 }, "violations": [ "account-already-initialized" ] }
    
```

### Autorización de transacciones

#### Entrada
Intenta autorizar una transacción para un comerciante en particular, cantidad y tiempo dado el estado de la cuenta y la última autorización actas.

#### Salida
El estado actual de la cuenta + cualquier violación de lógica de negocios.

#### Violaciones de lógica de negocios
Debe implementar las siguientes reglas, teniendo en cuenta *que aparecerán nuevas reglas* en el futuro:

1. El monto de la transacción no debe exceder el límite disponible: límite-insuficiente
2. No se debe aceptar ninguna transacción cuando la tarjeta no está activa: tarjeta-no-activa
3. No debe haber más de 3 transacciones en un intervalo de 2 minutos: intervalo-pequeño-alta-frecuencia
4. No debe haber más de 2 transacciones similares (misma cantidad y comerciante) en un intervalo de 2 minutos: transacción-duplicada

#### Ejemplos

* Dado que hay una cuenta con `activeCard: true` y `availableLimit: 100`:

```json
entrada
    { "transaction": { "merchant": "Burger King", "amount": 20, "time": "2019-02-13T10:00:00.000Z" } }
salida
    { "account": { "activeCard": true, "availableLimit": 80 }, "violations": [] }
```

* Dado que hay una cuenta con `activeCard: true` y `availableLimit: 80`:

```json
entrada
    { "transaction": { "merchant": "Habbib's", "amount": 90, "time": "2019-02-13T11:00:00.000Z" } }
salida
    { "account": { "activeCard": true, "availableLimit": 80 }, "violations": [ "insufficient-limit" ] }
```

## Uso

```bash
   curl -X POST -d '{ "account": { "id": "3bjk2h39", "activeCard": true, "availableLimit": 100 }, "violations": [] }' http://localhost:8080/autorizador/cuenta
```

```bash
   curl -X POST -d '{ "transaction": { "merchant": "Burger King", "amount": 20, "time": "2019-02-13T10:00:00.000Z" } }' http://localhost:8080/autorizador/autorizacion
```

