
# Protocols industrials


## EtherNet/IP (CIP)

És el protocol natiu de la plataforma Sysmac. Utilitza el protocol CIP (Common Industrial Protocol).

- Avantatges: És molt ràpid i permet treballar amb "Variable Names" (tags) directament, sense haver de fer servir adreces de memòria tipus D0 o H0.

- Ús des de PC: Hi ha llibreries com libplctag (Open Source) o drivers comercials de pagament (com Kepware o l'SDK de Omron).

https://github.com/libplctag/libplctag

## OPC UA (Servidor integrat)
Aquesta és, probablement, la millor opció per a una connexió amb un ordinador modern. El NX1P2 té un servidor OPC UA integrat al mateix hardware.

- Avantatges: És un estàndard industrial universal i segur (suporta certificats i xifrat). No necessites drivers propietaris a l'ordinador, només un client OPC UA (com UaExpert per testar o llibreries en Python/C#).

- Nota: Requereix una configuració prèvia des de Sysmac Studio per triar quines variables vols exposar.

## FINS (TCP o UDP)
És el protocol clàssic de Omron. Encara que és antic, el NX1P2 el manté per retrocompatibilitat.

- Avantatges: Molt senzill d'implementar si fas la teva pròpia aplicació (és un protocol de trama fixa bastant documentat).

- Inconvenient: Només pot accedir a zones de memòria clàssiques (com l'àrea E o D que hagis mapejat prèviament).

## Modbus TCP
El NX1P2 pot actuar tant com a Client com a Server Modbus TCP.

- Com funciona: No és "natiu" en el sentit que hagis de marcar una casella, sinó que s'utilitzen Function Blocks (FB) gratuïtes que Omron proporciona a la seva llibreria oficial.

- Ús: Ideal si l'ordinador ja té un software que parla Modbus de forma estàndard.
