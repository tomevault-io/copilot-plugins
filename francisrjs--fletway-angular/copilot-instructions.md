## fletway-angular

> Fletway es una plataforma digital (Web y Móvil) diseñada para conectar usuarios que necesitan transportar cargas medianas (muebles, paquetes) con conductores verificados (fleteros) que poseen vehículos de alta capacidad. El objetivo es optimizar el proceso de envíos bajo demanda, ofreciendo seguridad, seguimiento en tiempo real y precios competitivos.


# Fletway

Fletway es una plataforma digital (Web y Móvil) diseñada para conectar usuarios que necesitan transportar cargas medianas (muebles, paquetes) con conductores verificados (fleteros) que poseen vehículos de alta capacidad. El objetivo es optimizar el proceso de envíos bajo demanda, ofreciendo seguridad, seguimiento en tiempo real y precios competitivos.

## Stack Tecnológico y Arquitectura

El sistema utiliza una arquitectura MVC y se basa en servicios gestionados para reducir la complejidad del backend.  
 Frontend Web: Angular 17+ con TailwindCSS (Diseño responsivo).  
 Frontend Móvil: Flowbite (Base de código única para Android/iOS/Web).  
 Backend (Lógica de Negocio): Python Flask y Flask (API RESTful ligera para cálculos y orquestación).  
 Base de Datos y Auth: Supabase (PostgreSQL gestionado + Supabase Auth para gestión de sesiones).  
 Mapas: OpenStreetMap (API para renderizado y cálculo de rutas).

#Actores del Sistema

    Cliente (Usuario): Solicita fletes, recibe presupuestos, paga y califica.



    Fletero (Conductor): Recibe solicitudes cercanas, cotiza servicios, ejecuta el transporte y gestiona su perfil.



    Administrador: Gestión de usuarios, monitoreo y soporte (Backoffice).

# Módulos y Requisitos Funcionales (RF)

## Autenticación y Perfil

    Login/Registro: Autenticación mediante correo/contraseña usando Supabase Auth.

    Perfiles: Gestión de datos personales. Los fleteros deben tener un perfil verificado (documentación, vehículo).

## Solicitud de Fletes (Cliente)

    Crear Pedido (RF-PE-01): El cliente ingresa origen, destino (mapa), fecha/hora y características de la carga (incluyendo subida de fotos).



    Historial (RF-PE-02): Visualización de pedidos pasados y activos.

## Matchmaking y Presupuestos

    Listado de Solicitudes (RF-MM-01): El fletero ve una lista filtrada por ubicación, disponibilidad y tipo de carga.



    Cotización (RF-PR-02): El fletero envía un presupuesto (cotización) para una solicitud específica.

    Aceptación (RF-PR-03): El cliente recibe cotizaciones y puede aceptar una. Al aceptar, el estado cambia a "Confirmado" y se bloquean nuevas ofertas.

## Ejecución y Seguimiento

    Chat (RF-CS-01): Chat en tiempo real entre cliente y fletero habilitado tras confirmar el servicio.



    Mapa Interactivo (RF-IS-01): Visualización de la ruta planificada y ubicación (OpenStreetMap).

    Estados del Viaje: Pendiente -> Aceptado -> En Curso -> Finalizado.

## Cierre y Calificación

    Calificación (RF-CR-01): Sistema de 1 a 5 estrellas y reseña de texto. Solo habilitado cuando el viaje está "Finalizado".



    Pago: Procesamiento a través de Mercado Pago.

# Diseño e UI (Guía de Estilo)

Paleta de colores oficial:  
 Naranja (Principal): #FF6F00
Azul Grisáceo: #455A64
Blanco: #FFFFFF
Beige Claro: #FBE9E7

# Buenas practicas

Quiero que tengas buenas practicas en mente al momento de programar

1. Arquitectura de Carpetas: "Screaming Architecture"

En lugar de organizar por tipo de archivo (componentes, servicios), organizaremos por funcionalidad (Features). Esto hace que el código "grite" de qué trata (Fletes, Auth, Mapa) en lugar de qué tecnología usa.

Estructura sugerida para src/app:
Plaintext

src/app/
├── core/ # Singleton services, guards, interceptors, modelos globales
│ ├── auth/ # Servicio de SupabaseAuth, Guards
│ ├── interceptors/ # Manejo de errores HTTP, Tokens
│ └── models/ # Interfaces TS (Pedido, Usuario, Cotizacion)
├── shared/ # UI Components reutilizables (Botones, Inputs, Spinners)
│ ├── ui/
│ └── utils/ # Funciones de ayuda (formatDate, validadores)
├── features/ # Módulos principales del negocio (Lazy Loaded)
│ ├── auth/ # Login, Registro, Recuperar pass
│ ├── cliente/ # Dashboard cliente, Crear Pedido
│ ├── fletero/ # Dashboard fletero, Ver Ofertas
│ └── mapa/ # Componente de OpenStreetMap (Leaflet)
└── app.routes.ts # Rutas principales

2. Buenas Prácticas Clave (Angular 17+)
   A. Componentes Standalone (Adiós NgModules)

Ya no uses NgModules. Todos tus componentes deben ser standalone: true. Esto reduce el código repetitivo y facilita el Lazy Loading.
TypeScript

// features/mapa/mapa-view.component.ts
@Component({
selector: 'app-mapa-view',
standalone: true,
imports: [CommonModule], // Importa solo lo que necesites
templateUrl: './mapa-view.component.html'
})
export class MapaViewComponent {}

B. Uso de Signals para Gestión de Estado

Para Fletway, donde la ubicación del fletero cambia constantemente, los Signals son más eficientes que los BehaviorSubjects tradicionales de RxJS para el estado de la vista.

    Usa Signals para estado síncrono local (loading, formularios, visibilidad).

    Usa RxJS para eventos asíncronos complejos (peticiones HTTP, Websockets de Supabase).

Ejemplo: Servicio de Pedidos con Signals
TypeScript

// core/services/pedido.service.ts
import { Injectable, signal, inject } from '@angular/core';
import { SupabaseClient } from '@supabase/supabase-js';

@Injectable({ providedIn: 'root' })
export class PedidoService {
private supabase = inject(SupabaseClient); // Inyección moderna con inject()

// Signal que mantiene la lista de pedidos activos
public pedidosActivos = signal<Pedido[]>([]);

async cargarPedidos() {
const { data, error } = await this.supabase
.from('pedidos')
.select('\*')
.eq('estado', 'PENDIENTE');

    if (data) {
      this.pedidosActivos.set(data); // Actualiza el signal y la vista automáticamente
    }

}
}

C. Control Flow Syntax (Nuevo en HTML)

Deja de usar *ngIf y *ngFor. La nueva sintaxis es más rápida y legible.
HTML

@if (user()) {
<app-user-profile [user]="user()" />
} @else {
<button (click)="login()">Iniciar Sesión</button>
}

@for (pedido of pedidos(); track pedido.id) {

  <div class="card-pedido">
    {{ pedido.origen }} -> {{ pedido.destino }}
  </div>
} @empty {
  <p>No hay pedidos disponibles.</p>
}
dasdsadsadasd
3. Configuración del Stack (Tailwind + Supabase)

Usamos tailwind y supabase. 5. Reglas de componentes
En shared tenemos todos los componentes compartidos

- Popups
- Modales
- Botones
- Inputs
- Spinners
- Skeletons (componentes de carga)
- etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Francisrjs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
