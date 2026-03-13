# Entornos-7.5

## Fase 1: Análisis de Requisitos
```mermaid
graph LR
%% Actores
Member((Member))
Admin((Admin))


%% Definir límite del Sistema y Acciones
subgraph "Gym Class Reservation"
CU1([Reserve Gym Class])
CU2([Login])
CU3([Wait List])
CU4([Register New Class])
CU5([Cancel Session])


%% Relaciones especiales (include y extend)
CU1 -.->|&lt;&lt;include&gt;&gt;| CU2
CU3 -.->|&lt;&lt;extend&gt;&gt;| CU1
end
%% Rrelaciones Actor y Casos de Uso
Member --- CU1
Admin --- CU4
Admin --- CU5
```

# Fase 2: Diseño de la Interacción
### Diagrama de Secuencia
``` mermaid
sequenceDiagram
%% Numerado automatico para mejor seguimiento
autonumber

%% Actores y Participantes
actor Member 
participant WebInterface
participant ReservationManager
participant Database

%% Definir flujo
Member->>WebInterface: selectConfirmReservation
activate WebInterface
WebInterface->>ReservationManager: openManager
activate ReservationManager
ReservationManager->>Database: getData()
activate Database
Database-->>ReservationManager: retrieveData
deactivate Database
ReservationManager-->>WebInterface: reservationData
deactivate ReservationManager

%% Condiciones segun el resultado de buscar en la Base de Datos
alt isAvailable
WebInterface -->>Member: reservationAvailable
else isNotAvailable
WebInterface-->>Member: reservationUnavailable
end
deactivate WebInterface
```

### Diagrama de Comunicación
``` mermaid
graph LR
Member((:Member))
-- "1: selectConfirmReservation"
--> WebInterface((:WebInterface))
WebInterface
-- "1.1: openReservationManager()"
--> Manager((:ReservationManager))
Manager
-- "2: getData()"
--> Database((:Database))
```

# Fase 3: Lógica del Proceso
``` mermaid
stateDiagram-v2
%% Valida la reserva
[*] --> ReceiveRequest
%% Recibe la peticion y revisa los datos
ReceiveRequest --> CheckMemberData
%% Si tiene la cuota pagada (o no) decide la siguiente accion
state check_fee <<choice>>
CheckMemberData --> check_fee
check_fee --> ProcessReservation: [fee paid]
check_fee --> NotifyReservationIneligible: [fee not paid]
%% Si el socio ha pagado revisa el aforo
state check_capacity <<choice>>
ProcessReservation --> check_capacity
check_capacity --> AssignPlace: [capacity not full]
check_capacity --> NotifyNoPlaceAvailable : [full capacity]
%% Si hay aforo reserva la plaza
AssignPlace --> EmailConfirm: [place assigned]
%% Envia email
EmailConfirm --> [*]
```

## Fase 4: Ciclo de Vida del Objeto
``` mermaid
Diagrama de Estados
stateDiagram-v2
%% Draft: Está creada o creándose
[*] --> Draft
%%% Issued: Está creada y enviada
Draft --> Issued : send()
%% Confirmed: Está confirmada
Issued --> Confirmed : confirm()
%% Cancelled: Está cancelada
Issued --> Cancelled: cancel()
%% Realizada: Fue confirmada y el Socio se presentó
Confirmed --> CheckedIn: checkIn()
%% NotShownUp: Fue confirmada pero el Socio no se presentó
Confirmed --> NotShownUp: didNotShowUp()

%% Estados Finales
CheckedIn --> [*]
NotShownUp --> [*]
Cancelled --> [*]
```