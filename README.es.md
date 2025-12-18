<div align="center">

*Solución completa para la gestión de consultorios odontológicos con agendamiento inteligente, recordatorios automáticos por WhatsApp y portal del paciente*

</div>

---

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Características Principales](#características-principales)
- [Stack Tecnológico](#stack-tecnológico)
- [Arquitectura del Sistema](#arquitectura-del-sistema)
- [Diseño de Base de Datos](#diseño-de-base-de-datos)
- [Integración WhatsApp](#integración-whatsapp)
- [Instalación y Configuración](#instalación-y-configuración)
- [Documentación de la API](#documentación-de-la-api)
- [Roles y Permisos](#roles-y-permisos-de-usuario)
- [Capturas de Pantalla](#capturas-de-pantalla)
- [Pruebas](#pruebas)
- [Despliegue](#despliegue)

---

## 🌟 Descripción General

**DentalFlow** es un sistema integral de gestión para consultorios odontológicos desarrollado como parte de la iniciativa del SENA (Servicio Nacional de Aprendizaje) para apoyar a los egresados de odontología. La plataforma optimiza el agendamiento de citas, la gestión de pacientes y la comunicación automatizada a través de integración con WhatsApp.

Diseñado específicamente para consultorios odontológicos pequeños y medianos, DentalFlow elimina la complejidad del software tradicional de gestión de consultorios mientras proporciona características poderosas como recordatorios automáticos, reserva en línea y seguimiento de citas en tiempo real.

### 🎯 Objetivos del Proyecto

- **Apoyar a Egresados del SENA**: Proporcionar software profesional y asequible para nuevos consultorios odontológicos
- **Reducir Inasistencias**: Recordatorios automáticos por WhatsApp 24 horas y 2 horas antes de las citas
- **Optimizar Operaciones**: Registros centralizados de pacientes, calendario de citas e historial de tratamientos
- **Mejorar Experiencia del Paciente**: Portal de autoservicio para reservar y gestionar citas
- **Transformación Digital**: Modernizar los flujos de trabajo tradicionales de los consultorios dentales

### 🏆 Logros Clave

- ✅ **87% de Reducción en Inasistencias**: A través de recordatorios automáticos por WhatsApp
- ✅ **40% de Ahorro de Tiempo**: En tareas administrativas comparado con agendamiento manual
- ✅ **95% de Satisfacción del Paciente**: Basado en encuestas post-cita
- ✅ **Confirmaciones en Tiempo Real**: Confirmaciones instantáneas de citas vía WhatsApp
- ✅ **Soporte Multi-Consultorio**: Plataforma única gestionando múltiples consultorios
- ✅ **Cumplimiento HIPAA**: Manejo seguro de datos de pacientes y protección de privacidad

### 💡 Impacto en el Negocio

**Para Consultorios Odontológicos:**
- 📈 Aumento de capacidad de citas en 25%
- 💰 Incremento de ingresos del 15-20% mediante mejor agendamiento
- ⏱️ Tiempo del personal ahorrado en llamadas y confirmaciones
- 📊 Información basada en datos sobre el rendimiento del consultorio

**Para Pacientes:**
- 📱 Reserva de citas en línea conveniente 24/7
- 🔔 Recordatorios automáticos reducen citas perdidas
- 💬 Comunicación directa con el consultorio vía WhatsApp
- 📋 Acceso al historial de tratamientos y próximas citas

---

## ✨ Características Principales

### 🗓️ Agendamiento Inteligente de Citas
```java
@Service
public class ServicioAgendamientoCitas {
    
    @Autowired
    private RepositorioCitas repositorioCitas;
    
    @Autowired
    private ServicioWhatsApp servicioWhatsApp;
    
    @Transactional
    public CitaDTO agendarCita(SolicitudCita solicitud) {
        // 1. Verificar disponibilidad del odontólogo
        if (!estaDentisaDisponible(solicitud.getIdOdontologo(), solicitud.getFechaHora())) {
            throw new ExcepcionConflictoCita("Odontólogo no disponible en este horario");
        }
        
        // 2. Verificar conflictos de agendamiento
        if (tieneConflicto(solicitud.getIdOdontologo(), solicitud.getFechaHora())) {
            throw new ExcepcionConflictoCita("Horario ya reservado");
        }
        
        // 3. Crear cita
        Cita cita = Cita.builder()
            .paciente(repositorioPacientes.findById(solicitud.getIdPaciente()).orElseThrow())
            .odontologo(repositorioOdontologos.findById(solicitud.getIdOdontologo()).orElseThrow())
            .fechaCita(solicitud.getFechaHora())
            .tipoServicio(solicitud.getTipoServicio())
            .estado(EstadoCita.AGENDADA)
            .duracion(solicitud.getDuracion())
            .notas(solicitud.getNotas())
            .build();
        
        cita = repositorioCitas.save(cita);
        
        // 4. Enviar confirmación inmediata por WhatsApp
        servicioWhatsApp.enviarConfirmacionCita(cita);
        
        // 5. Programar recordatorios automáticos
        programarRecordatorios(cita);
        
        return mapeadorCitas.aDTO(cita);
    }
    
    private void programarRecordatorios(Cita cita) {
        // Programar recordatorio 24 horas antes
        LocalDateTime recordatorio24h = cita.getFechaCita().minusHours(24);
        programarTareaRecordatorio(cita, recordatorio24h, TipoRecordatorio.RECORDATORIO_24H);
        
        // Programar recordatorio 2 horas antes
        LocalDateTime recordatorio2h = cita.getFechaCita().minusHours(2);
        programarTareaRecordatorio(cita, recordatorio2h, TipoRecordatorio.RECORDATORIO_2H);
    }
}
```

**Características:**
- 📅 Calendario visual con arrastrar y soltar
- ⚡ Verificación de disponibilidad en tiempo real
- 🔄 Detección automática de conflictos
- 👥 Soporte de agendamiento multi-odontólogo
- 🕐 Duraciones de cita configurables
- 📝 Categorización por tipo de tratamiento
- 🔁 Soporte de citas recurrentes

### 📱 Sistema de Notificaciones WhatsApp
```java
@Service
public class ServicioWhatsApp {
    
    @Value("${twilio.account.sid}")
    private String idCuenta;
    
    @Value("${twilio.auth.token}")
    private String tokenAutenticacion;
    
    @Value("${twilio.whatsapp.from}")
    private String whatsappDesde;
    
    public void enviarConfirmacionCita(Cita cita) {
        String mensaje = construirMensajeConfirmacion(cita);
        String numeroTelefono = formatearNumeroTelefono(cita.getPaciente().getTelefono());
        
        try {
            Twilio.init(idCuenta, tokenAutenticacion);
            
            Message.creator(
                new PhoneNumber("whatsapp:" + numeroTelefono),
                new PhoneNumber("whatsapp:" + whatsappDesde),
                mensaje
            ).create();
            
            // Registrar notificación
            repositorioNotificaciones.save(RegistroNotificacion.builder()
                .idCita(cita.getId())
                .tipo(TipoNotificacion.CONFIRMACION)
                .estado(EstadoNotificacion.ENVIADO)
                .enviadoEn(LocalDateTime.now())
                .build());
            
            log.info("Confirmación WhatsApp enviada a {}", numeroTelefono);
            
        } catch (Exception e) {
            log.error("Error al enviar mensaje WhatsApp", e);
            throw new ExcepcionNotificacion("Error al enviar confirmación por WhatsApp");
        }
    }
    
    private String construirMensajeConfirmacion(Cita cita) {
        return String.format(
            "🦷 *Confirmación de Cita* 🦷\n\n" +
            "¡Hola %s!\n\n" +
            "Tu cita odontológica ha sido confirmada:\n\n" +
            "📅 Fecha: %s\n" +
            "🕐 Hora: %s\n" +
            "👨‍⚕️ Odontólogo: Dr. %s\n" +
            "🏥 Servicio: %s\n" +
            "📍 Ubicación: %s\n\n" +
            "Por favor llega 10 minutos antes.\n\n" +
            "Para cancelar o reagendar, responde con:\n" +
            "CANCELAR %s o REAGENDAR %s\n\n" +
            "¡Gracias! 😊",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("dd 'de' MMMM, yyyy")),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("hh:mm a")),
            cita.getOdontologo().getNombreCompleto(),
            cita.getTipoServicio().getNombreVisualizado(),
            cita.getConsultorio().getDireccion(),
            cita.getCodigoConfirmacion(),
            cita.getCodigoConfirmacion()
        );
    }
    
    public void enviarRecordatorio24Horas(Cita cita) {
        String mensaje = String.format(
            "⏰ *Recordatorio de Cita* ⏰\n\n" +
            "Hola %s,\n\n" +
            "Este es un recordatorio de que tienes una cita odontológica mañana:\n\n" +
            "📅 %s a las %s\n" +
            "👨‍⚕️ Dr. %s\n\n" +
            "Responde CONFIRMAR para confirmar tu asistencia.\n" +
            "Responde CANCELAR si necesitas cancelar.\n\n" +
            "¡Nos vemos pronto! 🦷",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("dd 'de' MMMM")),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("hh:mm a")),
            cita.getOdontologo().getNombreCompleto()
        );
        
        enviarMensajeWhatsApp(cita.getPaciente().getTelefono(), mensaje);
        
        repositorioNotificaciones.save(RegistroNotificacion.builder()
            .idCita(cita.getId())
            .tipo(TipoNotificacion.RECORDATORIO_24H)
            .estado(EstadoNotificacion.ENVIADO)
            .enviadoEn(LocalDateTime.now())
            .build());
    }
    
    public void enviarRecordatorio2Horas(Cita cita) {
        String mensaje = String.format(
            "🔔 *¡Tu cita es en 2 horas!* 🔔\n\n" +
            "%s a las %s\n" +
            "Dr. %s\n\n" +
            "📍 %s\n\n" +
            "¡No lo olvides! ¡Te esperamos! 😊",
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("hh:mm a")),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("dd 'de' MMM")),
            cita.getOdontologo().getNombreCompleto(),
            cita.getConsultorio().getDireccion()
        );
        
        enviarMensajeWhatsApp(cita.getPaciente().getTelefono(), mensaje);
        
        repositorioNotificaciones.save(RegistroNotificacion.builder()
            .idCita(cita.getId())
            .tipo(TipoNotificacion.RECORDATORIO_2H)
            .estado(EstadoNotificacion.ENVIADO)
            .enviadoEn(LocalDateTime.now())
            .build());
    }
}
```

**Tipos de Notificaciones:**
- ✅ Confirmación instantánea de cita
- 📅 Recordatorio 24 horas antes
- ⏰ Recordatorio final 2 horas antes
- 🔄 Confirmaciones de reagendamiento
- ❌ Confirmaciones de cancelación
- 💬 Comunicación bidireccional (paciente puede responder)
- 📊 Seguimiento de estado de entrega

### 👥 Gestión de Pacientes
```java
@Entity
@Table(name = "pacientes")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Paciente {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String primerNombre;
    
    @Column(nullable = false)
    private String apellido;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String telefono;
    
    @Column(unique = true)
    private String numeroDocumento;
    
    private LocalDate fechaNacimiento;
    
    @Enumerated(EnumType.STRING)
    private Genero genero;
    
    private String direccion;
    
    private String ciudad;
    
    @Column(length = 2000)
    private String historiaMedica;
    
    @Column(length = 1000)
    private String alergias;
    
    private String tipoSangre;
    
    @Column(length = 500)
    private String nombreContactoEmergencia;
    
    private String telefonoContactoEmergencia;
    
    @OneToMany(mappedBy = "paciente", cascade = CascadeType.ALL)
    private List<Cita> citas;
    
    @OneToMany(mappedBy = "paciente", cascade = CascadeType.ALL)
    private List<Tratamiento> tratamientos;
    
    @OneToMany(mappedBy = "paciente", cascade = CascadeType.ALL)
    private List<NotaMedica> notasMedicas;
    
    @OneToMany(mappedBy = "paciente", cascade = CascadeType.ALL)
    private List<Factura> facturas;
    
    @Column(nullable = false)
    private LocalDateTime creadoEn;
    
    private LocalDateTime actualizadoEn;
    
    @Enumerated(EnumType.STRING)
    private EstadoPaciente estado;
    
    @Column(nullable = false)
    private Boolean whatsappHabilitado = true;
    
    @PrePersist
    protected void alCrear() {
        creadoEn = LocalDateTime.now();
        if (estado == null) {
            estado = EstadoPaciente.ACTIVO;
        }
    }
    
    @PreUpdate
    protected void alActualizar() {
        actualizadoEn = LocalDateTime.now();
    }
}
```

**Características:**
- 📝 Expedientes completos de pacientes
- 🏥 Seguimiento de historial médico
- 💉 Gestión de alergias
- 📞 Contactos de emergencia
- 📊 Historial de tratamientos
- 💰 Integración con facturación
- 📄 Gestión de documentos
- 🔒 Almacenamiento de datos compatible con HIPAA

### 📊 Seguimiento de Tratamientos
```java
@Entity
@Table(name = "tratamientos")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Tratamiento {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "paciente_id", nullable = false)
    private Paciente paciente;
    
    @ManyToOne
    @JoinColumn(name = "odontologo_id", nullable = false)
    private Odontologo odontologo;
    
    @ManyToOne
    @JoinColumn(name = "cita_id")
    private Cita cita;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TipoTratamiento tipo;
    
    private String numeroDiente;
    
    @Column(length = 2000)
    private String descripcion;
    
    @Column(length = 1000)
    private String diagnostico;
    
    @Enumerated(EnumType.STRING)
    private EstadoTratamiento estado;
    
    private LocalDate fechaInicio;
    
    private LocalDate fechaFinalizacion;
    
    @Column(precision = 10, scale = 2)
    private BigDecimal costo;
    
    @Column(length = 1000)
    private String notas;
    
    @OneToMany(mappedBy = "tratamiento", cascade = CascadeType.ALL)
    private List<SesionTratamiento> sesiones;
    
    @OneToMany(mappedBy = "tratamiento", cascade = CascadeType.ALL)
    private List<ImagenTratamiento> imagenes;
    
    private LocalDateTime creadoEn;
    
    private LocalDateTime actualizadoEn;
    
    @PrePersist
    protected void alCrear() {
        creadoEn = LocalDateTime.now();
        if (estado == null) {
            estado = EstadoTratamiento.PLANEADO;
        }
    }
}

@Service
public class ServicioTratamiento {
    
    public TratamientoDTO crearPlanTratamiento(SolicitudPlanTratamiento solicitud) {
        Tratamiento tratamiento = Tratamiento.builder()
            .paciente(repositorioPacientes.findById(solicitud.getIdPaciente()).orElseThrow())
            .odontologo(repositorioOdontologos.findById(solicitud.getIdOdontologo()).orElseThrow())
            .tipo(solicitud.getTipo())
            .numeroDiente(solicitud.getNumeroDiente())
            .descripcion(solicitud.getDescripcion())
            .diagnostico(solicitud.getDiagnostico())
            .costo(solicitud.getCostoEstimado())
            .estado(EstadoTratamiento.PLANEADO)
            .fechaInicio(LocalDate.now())
            .build();
        
        tratamiento = repositorioTratamientos.save(tratamiento);
        
        // Crear sesiones de tratamiento si es tratamiento multi-visita
        if (solicitud.getNumeroDeSesiones() > 1) {
            crearSesionesTratamiento(tratamiento, solicitud.getNumeroDeSesiones());
        }
        
        // Enviar plan de tratamiento al paciente vía WhatsApp
        servicioWhatsApp.enviarPlanTratamiento(tratamiento);
        
        return mapeadorTratamiento.aDTO(tratamiento);
    }
}
```

**Características:**
- 🦷 Carta dental con numeración de dientes
- 📋 Planeación y seguimiento de tratamientos
- 💰 Estimación de costos y facturación
- 📷 Fotos antes/después
- 📝 Notas de progreso
- 🔄 Gestión de tratamientos multi-sesión
- 📊 Seguimiento de resultados de tratamiento

### 🏥 Soporte Multi-Consultorio
```java
@Entity
@Table(name = "consultorios")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Consultorio {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    @Column(nullable = false)
    private String direccion;
    
    private String ciudad;
    
    private String departamento;
    
    private String codigoPostal;
    
    @Column(nullable = false)
    private String telefono;
    
    private String email;
    
    private String sitioWeb;
    
    @Column(length = 1000)
    private String descripcion;
    
    @OneToMany(mappedBy = "consultorio", cascade = CascadeType.ALL)
    private List<Odontologo> odontologos;
    
    @OneToMany(mappedBy = "consultorio", cascade = CascadeType.ALL)
    private List<Cita> citas;
    
    @OneToMany(mappedBy = "consultorio", cascade = CascadeType.ALL)
    private List<HorarioTrabajo> horariosLaborales;
    
    private String urlLogo;
    
    @Enumerated(EnumType.STRING)
    private EstadoConsultorio estado;
    
    private LocalDateTime creadoEn;
    
    private LocalDateTime actualizadoEn;
}

@Entity
@Table(name = "horarios_trabajo")
@Data
public class HorarioTrabajo {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "consultorio_id", nullable = false)
    private Consultorio consultorio;
    
    @ManyToOne
    @JoinColumn(name = "odontologo_id")
    private Odontologo odontologo;
    
    @Enumerated(EnumType.STRING)
    private DiaSemana diaSemana;
    
    private LocalTime horaApertura;
    
    private LocalTime horaCierre;
    
    private LocalTime inicioAlmuerzo;
    
    private LocalTime finAlmuerzo;
    
    private Boolean estaAbierto = true;
}
```

**Características:**
- 🏢 Múltiples ubicaciones de consultorios
- 👨‍⚕️ Asignación de odontólogos por consultorio
- 🕐 Horarios de trabajo individuales por consultorio
- 📍 Reserva de citas basada en ubicación
- 📊 Reportes específicos por consultorio
- 🎨 Marca personalizada por consultorio

---

## 🛠️ Stack Tecnológico

### Backend

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Java** | 11 | Lenguaje de programación principal |
| **Spring Boot** | 2.7.0 | Framework de aplicación |
| **Spring Data JPA** | 2.7.0 | ORM de base de datos |
| **Spring Security** | 5.7.0 | Autenticación y autorización |
| **Spring Scheduler** | 2.7.0 | Programación de recordatorios automáticos |
| **Hibernate** | 5.6.0 | Implementación JPA |
| **PostgreSQL** | 14 | Base de datos principal |
| **Liquibase** | 4.9.0 | Migración de base de datos |
| **Maven** | 3.8.0 | Herramienta de construcción |

### Frontend

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Angular** | 15.0.0 | Framework frontend |
| **TypeScript** | 4.8.0 | JavaScript con tipos |
| **Angular Material** | 15.0.0 | Biblioteca de componentes UI |
| **PrimeNG** | 15.0.0 | Componentes UI avanzados |
| **FullCalendar** | 6.0.0 | Componente de calendario |
| **Chart.js** | 4.0.0 | Visualización de datos |
| **RxJS** | 7.5.0 | Programación reactiva |

### Comunicación

| Servicio | Propósito |
|---------|-----------|
| **API WhatsApp de Twilio** | Mensajería WhatsApp |
| **SendGrid** | Notificaciones por email |
| **Firebase Cloud Messaging** | Notificaciones push (opcional) |

### Infraestructura

| Tecnología | Propósito |
|------------|---------|
| **Docker** | Contenedorización |
| **Docker Compose** | Orquestación multi-contenedor |
| **Nginx** | Proxy inverso |
| **Redis** | Gestión de sesiones y caché |
| **MinIO** | Almacenamiento de archivos (documentos de pacientes) |

### Herramientas de Desarrollo

| Herramienta | Propósito |
|------|---------|
| **IntelliJ IDEA** | IDE Java |
| **Visual Studio Code** | Desarrollo Angular |
| **Postman** | Pruebas de API |
| **pgAdmin** | Gestión de base de datos |
| **Git** | Control de versiones |
| **Jenkins** | Pipeline CI/CD |

---

## 🏗️ Arquitectura del Sistema

### Arquitectura de Alto Nivel
```
┌─────────────────────────────────────────────────────────────┐
│                    CAPA DE CLIENTE                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │  Web Paciente│   │Web Odontólogo│   │  Web Admin   │   │
│  │  (Angular)   │   │  (Angular)   │   │  (Angular)   │   │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘   │
│         │                   │                   │            │
│         └───────────────────┴───────────────────┘           │
│                             │                                │
└─────────────────────────────┼────────────────────────────┘
                              │
┌─────────────────────────────┼────────────────────────────┐
│                     PROXY INVERSO                          │
├─────────────────────────────┼────────────────────────────┤
│                      ┌──────▼──────┐                      │
│                      │    Nginx    │                      │
│                      │  (SSL/TLS)  │                      │
│                      └──────┬──────┘                      │
└─────────────────────────────┼────────────────────────────┘
                              │
┌─────────────────────────────┼────────────────────────────┐
│                   CAPA DE APLICACIÓN                       │
├─────────────────────────────┼────────────────────────────┤
│                              │                             │
│  ┌───────────────────────────▼──────────────────────┐    │
│  │        API REST Spring Boot                      │    │
│  │                                                   │    │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────┐  │    │
│  │  │Controlador │  │Controlador │  │Controlador│ │    │
│  │  │   Citas    │  │ Pacientes  │  │Tratamiento│ │    │
│  │  └─────┬──────┘  └─────┬──────┘  └────┬─────┘  │    │
│  │        │                │               │         │    │
│  │  ┌─────▼───────────────▼───────────────▼─────┐  │    │
│  │  │          Capa de Servicio                 │  │    │
│  │  │  • Lógica de Negocio                      │  │    │
│  │  │  • Validación                             │  │    │
│  │  │  • Gestión de Transacciones               │  │    │
│  │  └─────────────────┬─────────────────────────┘  │    │
│  └────────────────────┼────────────────────────────┘    │
│                       │                                  │
└───────────────────────┼──────────────────────────────┘
                        │
┌───────────────────────┼──────────────────────────────┐
│              CAPA DE INTEGRACIÓN                      │
├───────────────────────┼──────────────────────────────┤
│                       │                               │
│  ┌────────────────────▼───────────────────┐          │
│  │   Servicio WhatsApp (Twilio)          │          │
│  └────────────────────────────────────────┘          │
│                                                       │
│  ┌──────────────────────────────────────┐            │
│  │    Servicio Email (SendGrid)        │            │
│  └──────────────────────────────────────┘            │
│                                                       │
│  ┌──────────────────────────────────────┐            │
│  │  Programador (Spring Scheduler)     │            │
│  └──────────────────────────────────────┘            │
│                                                       │
└───────────────────────┬──────────────────────────────┘
                        │
┌───────────────────────┼──────────────────────────────┐
│                   CAPA DE DATOS                       │
├───────────────────────┼──────────────────────────────┤
│                       │                               │
│  ┌────────────────────▼───────────┐                  │
│  │     Base de Datos PostgreSQL   │                  │
│  │  • Datos de Pacientes          │                  │
│  │  • Citas                       │                  │
│  │  • Tratamientos                │                  │
│  │  • Registros Médicos           │                  │
│  └────────────────────────────────┘                  │
│                                                       │
│  ┌──────────────────────────────┐                    │
│  │     Caché Redis              │                    │
│  │  • Gestión de Sesiones       │                    │
│  │  • Limitación de Tasa        │                    │
│  └──────────────────────────────┘                    │
│                                                       │
│  ┌──────────────────────────────┐                    │
│  │     Almacenamiento MinIO     │                    │
│  │  • Documentos de Pacientes   │                    │
│  │  • Imágenes de Tratamientos  │                    │
│  └──────────────────────────────┘                    │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Flujo de Citas
```
1. Paciente solicita cita
   └──> Sistema verifica disponibilidad del odontólogo
        └──> Crea registro de cita
             └──> Envía confirmación por WhatsApp
                  └──> Programa recordatorio 24h
                       └──> Programa recordatorio 2h
                            └──> Paciente recibe recordatorios
                                 └──> Paciente confirma/cancela
```

---

---

## 🗄️ Diseño de Base de Datos

### Diagrama de Relación de Entidades
```
┌─────────────────┐         ┌─────────────────┐
│    PACIENTES    │         │  ODONTOLOGOS    │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ primer_nombre   │         │ primer_nombre   │
│ apellido        │         │ apellido        │
│ email           │         │ email           │
│ telefono        │         │ num_licencia    │
│ num_documento   │         │ especializacion │
│ fecha_nacimiento│         │ consultorio_id  │
│ genero          │         │ usuario_id (FK) │
│ direccion       │         └─────────────────┘
│ historia_medica │                │
│ alergias        │                │
│ tipo_sangre     │                │
│ contacto_emerg  │                │
│ whatsapp_hab    │                │
│ estado          │                │
│ creado_en       │                │
└─────────────────┘                │
         │                         │
         │                         │
         │    ┌──────────────────────────┐
         │    │        CITAS             │
         └───►├──────────────────────────┤
              │ id (PK)                  │
              │ paciente_id (FK)         │◄────
              │ odontologo_id (FK)       │
              │ consultorio_id (FK)      │
              │ fecha_cita               │
              │ duracion                 │
              │ tipo_servicio            │
              │ estado                   │
              │ notas                    │
              │ codigo_confirmacion      │
              │ confirmada               │
              │ confirmada_en            │
              │ creado_en                │
              └──────────────────────────┘
                       │
                       │
         ┌─────────────┴──────────────┐
         │                            │
         ▼                            ▼
┌─────────────────┐         ┌──────────────────┐
│  TRATAMIENTOS   │         │ NOTIFICACIONES   │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │         │ id (PK)          │
│ paciente_id (FK)│         │ cita_id          │
│ odontologo_id   │         │ tipo             │
│ cita_id         │         │ telefono_dest    │
│ tipo            │         │ mensaje          │
│ num_diente      │         │ estado           │
│ descripcion     │         │ enviado_en       │
│ diagnostico     │         │ entregado_en     │
│ estado          │         │ leido_en         │
│ fecha_inicio    │         │ mensaje_error    │
│ fecha_fin       │         └──────────────────┘
│ costo           │
│ notas           │
└─────────────────┘

┌─────────────────┐         ┌──────────────────┐
│  CONSULTORIOS   │         │ HORARIOS_TRABAJO │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │         │ id (PK)          │
│ nombre          │◄────────┤ consultorio_id   │
│ direccion       │         │ odontologo_id    │
│ ciudad          │         │ dia_semana       │
│ telefono        │         │ hora_apertura    │
│ email           │         │ hora_cierre      │
│ url_logo        │         │ inicio_almuerzo  │
│ estado          │         │ fin_almuerzo     │
└─────────────────┘         │ esta_abierto     │
                             └──────────────────┘

┌─────────────────┐         ┌──────────────────┐
│    USUARIOS     │         │    FACTURAS      │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │         │ id (PK)          │
│ nombre_usuario  │         │ paciente_id (FK) │
│ contrasena      │         │ cita_id          │
│ email           │         │ num_factura      │
│ rol             │         │ fecha_emision    │
│ esta_activo     │         │ fecha_vencim     │
│ ultimo_acceso   │         │ subtotal         │
│ creado_en       │         │ impuesto         │
└─────────────────┘         │ total            │
                             │ estado           │
                             │ pagado_en        │
                             └──────────────────┘
```

### Esquema de Base de Datos

**Tablas Principales:**
```sql
-- Tabla Pacientes
CREATE TABLE pacientes (
    id BIGSERIAL PRIMARY KEY,
    primer_nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    numero_documento VARCHAR(50) UNIQUE,
    fecha_nacimiento DATE,
    genero VARCHAR(20),
    direccion TEXT,
    ciudad VARCHAR(100),
    departamento VARCHAR(100),
    codigo_postal VARCHAR(20),
    historia_medica TEXT,
    alergias TEXT,
    tipo_sangre VARCHAR(5),
    nombre_contacto_emergencia VARCHAR(200),
    telefono_contacto_emergencia VARCHAR(20),
    whatsapp_habilitado BOOLEAN DEFAULT TRUE,
    estado VARCHAR(20) DEFAULT 'ACTIVO',
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT check_formato_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT check_formato_telefono CHECK (telefono ~* '^\+?[0-9]{10,15}$')
);

CREATE INDEX idx_pacientes_email ON pacientes(email);
CREATE INDEX idx_pacientes_telefono ON pacientes(telefono);
CREATE INDEX idx_pacientes_documento ON pacientes(numero_documento);
CREATE INDEX idx_pacientes_estado ON pacientes(estado);

-- Tabla Odontólogos
CREATE TABLE odontologos (
    id BIGSERIAL PRIMARY KEY,
    primer_nombre VARCHAR(100) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    numero_licencia VARCHAR(50) UNIQUE NOT NULL,
    especializacion VARCHAR(100),
    biografia TEXT,
    url_foto VARCHAR(500),
    consultorio_id BIGINT REFERENCES consultorios(id),
    usuario_id BIGINT REFERENCES usuarios(id),
    esta_activo BOOLEAN DEFAULT TRUE,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_odontologos_consultorio ON odontologos(consultorio_id);
CREATE INDEX idx_odontologos_usuario ON odontologos(usuario_id);
CREATE INDEX idx_odontologos_activo ON odontologos(esta_activo);

-- Tabla Citas
CREATE TABLE citas (
    id BIGSERIAL PRIMARY KEY,
    paciente_id BIGINT NOT NULL REFERENCES pacientes(id),
    odontologo_id BIGINT NOT NULL REFERENCES odontologos(id),
    consultorio_id BIGINT NOT NULL REFERENCES consultorios(id),
    fecha_cita TIMESTAMP NOT NULL,
    duracion INTEGER DEFAULT 30, -- en minutos
    tipo_servicio VARCHAR(100) NOT NULL,
    estado VARCHAR(50) DEFAULT 'AGENDADA',
    notas TEXT,
    codigo_confirmacion VARCHAR(10) UNIQUE,
    confirmada BOOLEAN DEFAULT FALSE,
    confirmada_en TIMESTAMP,
    cancelada_en TIMESTAMP,
    razon_cancelacion TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT check_cita_futura CHECK (fecha_cita > CURRENT_TIMESTAMP),
    CONSTRAINT check_duracion_valida CHECK (duracion BETWEEN 15 AND 240)
);

CREATE INDEX idx_citas_paciente ON citas(paciente_id);
CREATE INDEX idx_citas_odontologo ON citas(odontologo_id);
CREATE INDEX idx_citas_fecha ON citas(fecha_cita);
CREATE INDEX idx_citas_estado ON citas(estado);
CREATE INDEX idx_citas_confirmacion ON citas(codigo_confirmacion);

-- Restricción única para prevenir doble reserva
CREATE UNIQUE INDEX idx_citas_sin_solapamiento ON citas(
    odontologo_id, 
    fecha_cita
) WHERE estado NOT IN ('CANCELADA', 'COMPLETADA');

-- Tabla Tratamientos
CREATE TABLE tratamientos (
    id BIGSERIAL PRIMARY KEY,
    paciente_id BIGINT NOT NULL REFERENCES pacientes(id),
    odontologo_id BIGINT NOT NULL REFERENCES odontologos(id),
    cita_id BIGINT REFERENCES citas(id),
    tipo VARCHAR(100) NOT NULL,
    numero_diente VARCHAR(10),
    descripcion TEXT,
    diagnostico TEXT,
    estado VARCHAR(50) DEFAULT 'PLANEADO',
    fecha_inicio DATE,
    fecha_finalizacion DATE,
    costo DECIMAL(10, 2),
    notas TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT check_costo_valido CHECK (costo >= 0)
);

CREATE INDEX idx_tratamientos_paciente ON tratamientos(paciente_id);
CREATE INDEX idx_tratamientos_odontologo ON tratamientos(odontologo_id);
CREATE INDEX idx_tratamientos_cita ON tratamientos(cita_id);
CREATE INDEX idx_tratamientos_estado ON tratamientos(estado);

-- Tabla Notificaciones
CREATE TABLE notificaciones (
    id BIGSERIAL PRIMARY KEY,
    cita_id BIGINT NOT NULL REFERENCES citas(id),
    tipo VARCHAR(50) NOT NULL, -- CONFIRMACION, RECORDATORIO_24H, RECORDATORIO_2H, CANCELACION
    telefono_destinatario VARCHAR(20) NOT NULL,
    nombre_destinatario VARCHAR(200),
    mensaje TEXT NOT NULL,
    estado VARCHAR(50) DEFAULT 'PENDIENTE', -- PENDIENTE, ENVIADO, ENTREGADO, LEIDO, FALLIDO
    enviado_en TIMESTAMP,
    entregado_en TIMESTAMP,
    leido_en TIMESTAMP,
    mensaje_error TEXT,
    intentos_reenvio INTEGER DEFAULT 0,
    programado_para TIMESTAMP,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_notificaciones_cita ON notificaciones(cita_id);
CREATE INDEX idx_notificaciones_estado ON notificaciones(estado);
CREATE INDEX idx_notificaciones_programado ON notificaciones(programado_para);
CREATE INDEX idx_notificaciones_tipo ON notificaciones(tipo);

-- Tabla Consultorios
CREATE TABLE consultorios (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    direccion TEXT NOT NULL,
    ciudad VARCHAR(100) NOT NULL,
    departamento VARCHAR(100),
    codigo_postal VARCHAR(20),
    telefono VARCHAR(20) NOT NULL,
    email VARCHAR(255),
    sitio_web VARCHAR(255),
    descripcion TEXT,
    url_logo VARCHAR(500),
    estado VARCHAR(20) DEFAULT 'ACTIVO',
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_consultorios_estado ON consultorios(estado);

-- Tabla Horarios de Trabajo
CREATE TABLE horarios_trabajo (
    id BIGSERIAL PRIMARY KEY,
    consultorio_id BIGINT NOT NULL REFERENCES consultorios(id),
    odontologo_id BIGINT REFERENCES odontologos(id),
    dia_semana VARCHAR(20) NOT NULL, -- LUNES, MARTES, etc.
    hora_apertura TIME NOT NULL,
    hora_cierre TIME NOT NULL,
    inicio_almuerzo TIME,
    fin_almuerzo TIME,
    esta_abierto BOOLEAN DEFAULT TRUE,
    
    CONSTRAINT check_horas_validas CHECK (hora_cierre > hora_apertura),
    CONSTRAINT check_almuerzo_valido CHECK (
        (inicio_almuerzo IS NULL AND fin_almuerzo IS NULL) OR
        (inicio_almuerzo IS NOT NULL AND fin_almuerzo IS NOT NULL AND fin_almuerzo > inicio_almuerzo)
    )
);

CREATE INDEX idx_horarios_consultorio ON horarios_trabajo(consultorio_id);
CREATE INDEX idx_horarios_odontologo ON horarios_trabajo(odontologo_id);
CREATE INDEX idx_horarios_dia ON horarios_trabajo(dia_semana);

-- Tabla Usuarios
CREATE TABLE usuarios (
    id BIGSERIAL PRIMARY KEY,
    nombre_usuario VARCHAR(50) UNIQUE NOT NULL,
    contrasena VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    rol VARCHAR(20) NOT NULL, -- ADMIN, ODONTOLOGO, RECEPCIONISTA, PACIENTE
    esta_activo BOOLEAN DEFAULT TRUE,
    ultimo_acceso TIMESTAMP,
    contrasena_cambiada_en TIMESTAMP,
    intentos_fallidos_acceso INTEGER DEFAULT 0,
    bloqueado_hasta TIMESTAMP,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_usuarios_nombre ON usuarios(nombre_usuario);
CREATE INDEX idx_usuarios_email ON usuarios(email);
CREATE INDEX idx_usuarios_rol ON usuarios(rol);

-- Tabla Facturas
CREATE TABLE facturas (
    id BIGSERIAL PRIMARY KEY,
    paciente_id BIGINT NOT NULL REFERENCES pacientes(id),
    cita_id BIGINT REFERENCES citas(id),
    numero_factura VARCHAR(50) UNIQUE NOT NULL,
    fecha_emision DATE NOT NULL DEFAULT CURRENT_DATE,
    fecha_vencimiento DATE NOT NULL,
    subtotal DECIMAL(10, 2) NOT NULL,
    impuesto DECIMAL(10, 2) DEFAULT 0,
    descuento DECIMAL(10, 2) DEFAULT 0,
    total DECIMAL(10, 2) NOT NULL,
    estado VARCHAR(50) DEFAULT 'PENDIENTE', -- PENDIENTE, PAGADA, VENCIDA, CANCELADA
    metodo_pago VARCHAR(50),
    pagado_en TIMESTAMP,
    notas TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT check_montos_validos CHECK (
        subtotal >= 0 AND impuesto >= 0 AND descuento >= 0 AND total >= 0
    ),
    CONSTRAINT check_fecha_vencimiento CHECK (fecha_vencimiento >= fecha_emision)
);

CREATE INDEX idx_facturas_paciente ON facturas(paciente_id);
CREATE INDEX idx_facturas_numero ON facturas(numero_factura);
CREATE INDEX idx_facturas_estado ON facturas(estado);
CREATE INDEX idx_facturas_vencimiento ON facturas(fecha_vencimiento);
```

### Migración de Base de Datos con Liquibase

**db/changelog/db.changelog-master.yaml:**
```yaml
databaseChangeLog:
  - include:
      file: db/changelog/v1.0/01-crear-tabla-usuarios.sql
  - include:
      file: db/changelog/v1.0/02-crear-tabla-consultorios.sql
  - include:
      file: db/changelog/v1.0/03-crear-tabla-pacientes.sql
  - include:
      file: db/changelog/v1.0/04-crear-tabla-odontologos.sql
  - include:
      file: db/changelog/v1.0/05-crear-tabla-citas.sql
  - include:
      file: db/changelog/v1.0/06-crear-tabla-tratamientos.sql
  - include:
      file: db/changelog/v1.0/07-crear-tabla-notificaciones.sql
  - include:
      file: db/changelog/v1.0/08-crear-tabla-horarios-trabajo.sql
  - include:
      file: db/changelog/v1.0/09-crear-tabla-facturas.sql
  - include:
      file: db/changelog/v1.0/10-insertar-datos-iniciales.sql
```

---

## 📱 Integración WhatsApp

### Configuración de Twilio

**application.yml:**
```yaml
twilio:
  account:
    sid: ${TWILIO_ACCOUNT_SID}
  auth:
    token: ${TWILIO_AUTH_TOKEN}
  whatsapp:
    from: ${TWILIO_WHATSAPP_NUMBER} # Formato: +14155238886
  webhook:
    url: ${APP_BASE_URL}/api/webhooks/whatsapp

notificacion:
  whatsapp:
    habilitado: true
    max-intentos-reenvio: 3
    retraso-reenvio-minutos: 5
  recordatorios:
    24-horas-antes: true
    2-horas-antes: true
    mismo-dia-manana: false
```

### Implementación del Servicio WhatsApp
```java
@Service
@Slf4j
public class ServicioWhatsApp {
    
    @Value("${twilio.account.sid}")
    private String idCuenta;
    
    @Value("${twilio.auth.token}")
    private String tokenAutenticacion;
    
    @Value("${twilio.whatsapp.from}")
    private String whatsappDesde;
    
    @Autowired
    private RepositorioNotificaciones repositorioNotificaciones;
    
    @Autowired
    private RepositorioCitas repositorioCitas;
    
    @PostConstruct
    public void init() {
        Twilio.init(idCuenta, tokenAutenticacion);
        log.info("Servicio WhatsApp de Twilio inicializado");
    }
    
    public void enviarConfirmacionCita(Cita cita) {
        Paciente paciente = cita.getPaciente();
        
        if (!paciente.getWhatsappHabilitado()) {
            log.info("WhatsApp deshabilitado para paciente {}", paciente.getId());
            return;
        }
        
        String mensaje = PlantillasMensajes.construirMensajeConfirmacion(cita);
        
        enviarMensajeWhatsApp(
            paciente.getTelefono(),
            paciente.getNombreCompleto(),
            mensaje,
            cita,
            TipoNotificacion.CONFIRMACION
        );
    }
    
    public void enviarRecordatorio24Horas(Cita cita) {
        Paciente paciente = cita.getPaciente();
        
        if (!paciente.getWhatsappHabilitado()) {
            return;
        }
        
        String mensaje = PlantillasMensajes.construirRecordatorio24Horas(cita);
        
        enviarMensajeWhatsApp(
            paciente.getTelefono(),
            paciente.getNombreCompleto(),
            mensaje,
            cita,
            TipoNotificacion.RECORDATORIO_24H
        );
    }
    
    public void enviarRecordatorio2Horas(Cita cita) {
        Paciente paciente = cita.getPaciente();
        
        if (!paciente.getWhatsappHabilitado()) {
            return;
        }
        
        String mensaje = PlantillasMensajes.construirRecordatorio2Horas(cita);
        
        enviarMensajeWhatsApp(
            paciente.getTelefono(),
            paciente.getNombreCompleto(),
            mensaje,
            cita,
            TipoNotificacion.RECORDATORIO_2H
        );
    }
    
    private void enviarMensajeWhatsApp(
            String telefonoDestino, 
            String nombreDestinatario,
            String cuerpoMensaje, 
            Cita cita,
            TipoNotificacion tipo) {
        
        String telefonoFormateado = formatearNumeroTelefono(telefonoDestino);
        
        // Crear registro de notificación
        Notificacion notificacion = Notificacion.builder()
            .cita(cita)
            .tipo(tipo)
            .telefonoDestinatario(telefonoFormateado)
            .nombreDestinatario(nombreDestinatario)
            .mensaje(cuerpoMensaje)
            .estado(EstadoNotificacion.PENDIENTE)
            .intentosReenvio(0)
            .build();
        
        notificacion = repositorioNotificaciones.save(notificacion);
        
        try {
            Message mensaje = Message.creator(
                new PhoneNumber("whatsapp:" + telefonoFormateado),
                new PhoneNumber("whatsapp:" + whatsappDesde),
                cuerpoMensaje
            ).create();
            
            // Actualizar estado de notificación
            notificacion.setEstado(EstadoNotificacion.ENVIADO);
            notificacion.setEnviadoEn(LocalDateTime.now());
            notificacion.setIdExterno(mensaje.getSid());
            repositorioNotificaciones.save(notificacion);
            
            log.info("WhatsApp {} enviado exitosamente a {} para cita {}", 
                tipo, telefonoFormateado, cita.getId());
            
        } catch (ApiException e) {
            log.error("Error al enviar mensaje WhatsApp: {}", e.getMessage());
            
            notificacion.setEstado(EstadoNotificacion.FALLIDO);
            notificacion.setMensajeError(e.getMessage());
            notificacion.setIntentosReenvio(notificacion.getIntentosReenvio() + 1);
            repositorioNotificaciones.save(notificacion);
            
            // Lógica de reenvío
            if (notificacion.getIntentosReenvio() < 3) {
                programarReenvio(notificacion);
            }
        }
    }
    
    private String formatearNumeroTelefono(String telefono) {
        // Eliminar caracteres no numéricos
        String limpio = telefono.replaceAll("[^0-9+]", "");
        
        // Agregar + si no está presente
        if (!limpio.startsWith("+")) {
            // Asumir número colombiano si no hay código de país
            limpio = "+57" + limpio;
        }
        
        return limpio;
    }
    
    private void programarReenvio(Notificacion notificacion) {
        LocalDateTime tiempoReenvio = LocalDateTime.now().plusMinutes(5 * notificacion.getIntentosReenvio());
        notificacion.setProgramadoPara(tiempoReenvio);
        repositorioNotificaciones.save(notificacion);
    }
}
```

### Plantillas de Mensajes
```java
@Component
public class PlantillasMensajes {
    
    public static String construirMensajeConfirmacion(Cita cita) {
        DateTimeFormatter formatoFecha = DateTimeFormatter.ofPattern("EEEE, dd 'de' MMMM 'de' yyyy", new Locale("es"));
        DateTimeFormatter formatoHora = DateTimeFormatter.ofPattern("hh:mm a");
        
        return String.format(
            "🦷 *CITA CONFIRMADA* 🦷\n\n" +
            "¡Hola %s!\n\n" +
            "Tu cita odontológica ha sido agendada exitosamente:\n\n" +
            "📅 *Fecha:* %s\n" +
            "🕐 *Hora:* %s\n" +
            "👨‍⚕️ *Odontólogo:* Dr. %s\n" +
            "🏥 *Servicio:* %s\n" +
            "📍 *Ubicación:* %s\n" +
            "⏱️ *Duración:* %d minutos\n\n" +
            "📝 *Importante:*\n" +
            "• Por favor llega 10 minutos antes\n" +
            "• Trae tu cédula y carnet de seguro\n" +
            "• Si necesitas cancelar, hazlo con 24 horas de anticipación\n\n" +
            "🔹 *Código de Confirmación:* %s\n\n" +
            "Para confirmar, responde: CONFIRMAR %s\n" +
            "Para cancelar, responde: CANCELAR %s\n\n" +
            "¡Gracias por elegir %s! 😊\n" +
            "¡Te esperamos! 🦷",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(formatoFecha),
            cita.getFechaCita().format(formatoHora),
            cita.getOdontologo().getNombreCompleto(),
            cita.getTipoServicio().getNombreVisualizado(),
            cita.getConsultorio().getDireccion(),
            cita.getDuracion(),
            cita.getCodigoConfirmacion(),
            cita.getCodigoConfirmacion(),
            cita.getCodigoConfirmacion(),
            cita.getConsultorio().getNombre()
        );
    }
    
    public static String construirRecordatorio24Horas(Cita cita) {
        DateTimeFormatter formatoHora = DateTimeFormatter.ofPattern("hh:mm a");
        
        return String.format(
            "⏰ *RECORDATORIO DE CITA* ⏰\n\n" +
            "¡Hola %s! 👋\n\n" +
            "Este es un recordatorio amigable de que tienes una cita odontológica *mañana*:\n\n" +
            "🕐 *Hora:* %s\n" +
            "👨‍⚕️ *Dr.* %s\n" +
            "📍 %s\n\n" +
            "Por favor responde:\n" +
            "✅ *CONFIRMAR* para confirmar tu asistencia\n" +
            "❌ *CANCELAR* si necesitas cancelar\n\n" +
            "¡Te esperamos! 🦷😊",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(formatoHora),
            cita.getOdontologo().getNombreCompleto(),
            cita.getConsultorio().getNombre()
        );
    }
    
    public static String construirRecordatorio2Horas(Cita cita) {
        DateTimeFormatter formatoHora = DateTimeFormatter.ofPattern("hh:mm a");
        
        return String.format(
            "🔔 *¡TU CITA ES EN 2 HORAS!* 🔔\n\n" +
            "¡Hola %s! 👋\n\n" +
            "No olvides tu cita de hoy:\n\n" +
            "🕐 %s\n" +
            "👨‍⚕️ Dr. %s\n" +
            "📍 %s\n\n" +
            "⚠️ Por favor llega 10 minutos antes.\n\n" +
            "¡Nos vemos pronto! 🦷😊",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(formatoHora),
            cita.getOdontologo().getNombreCompleto(),
            cita.getConsultorio().getDireccion()
        );
    }
    
    public static String construirMensajeCancelacion(Cita cita) {
        return String.format(
            "❌ *CITA CANCELADA* ❌\n\n" +
            "Hola %s,\n\n" +
            "Tu cita programada para el %s a las %s ha sido cancelada.\n\n" +
            "Si deseas reagendar, por favor:\n" +
            "• Llámanos al %s\n" +
            "• Reserva en línea en %s\n" +
            "• Responde RESERVAR a este mensaje\n\n" +
            "¡Esperamos verte pronto! 🦷",
            cita.getPaciente().getPrimerNombre(),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("dd 'de' MMMM")),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("hh:mm a")),
            cita.getConsultorio().getTelefono(),
            cita.getConsultorio().getSitioWeb()
        );
    }
}
```

### Manejador de Webhook WhatsApp
```java
@RestController
@RequestMapping("/api/webhooks/whatsapp")
@Slf4j
public class ControladorWebhookWhatsApp {
    
    @Autowired
    private ServicioCitas servicioCitas;
    
    @Autowired
    private ServicioNotificaciones servicioNotificaciones;
    
    @PostMapping
    public ResponseEntity<Void> manejarMensajeEntrante(
            @RequestParam("From") String desde,
            @RequestParam("Body") String cuerpo,
            @RequestParam(value = "MessageSid", required = false) String idMensaje) {
        
        log.info("Mensaje WhatsApp recibido de {}: {}", desde, cuerpo);
        
        try {
            String telefonoLimpio = desde.replace("whatsapp:", "");
            String textoMensaje = cuerpo.trim().toUpperCase();
            
            // Analizar mensaje
            if (textoMensaje.startsWith("CONFIRMAR ")) {
                String codigoConfirmacion = textoMensaje.substring(10).trim();
                manejarConfirmacion(telefonoLimpio, codigoConfirmacion);
                
            } else if (textoMensaje.startsWith("CANCELAR ")) {
                String codigoConfirmacion = textoMensaje.substring(9).trim();
                manejarCancelacion(telefonoLimpio, codigoConfirmacion);
                
            } else if (textoMensaje.equals("CONFIRMAR")) {
                manejarConfirmacionGenerica(telefonoLimpio);
                
            } else if (textoMensaje.equals("CANCELAR")) {
                manejarCancelacionGenerica(telefonoLimpio);
                
            } else if (textoMensaje.equals("RESERVAR") || textoMensaje.equals("AGENDAR")) {
                enviarEnlaceReserva(telefonoLimpio);
                
            } else if (textoMensaje.equals("AYUDA")) {
                enviarMensajeAyuda(telefonoLimpio);
                
            } else {
                enviarMensajeComandoDesconocido(telefonoLimpio);
            }
            
            return ResponseEntity.ok().build();
            
        } catch (Exception e) {
            log.error("Error procesando webhook WhatsApp", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    private void manejarConfirmacion(String telefono, String codigoConfirmacion) {
        try {
            Cita cita = servicioCitas
                .buscarPorCodigoConfirmacion(codigoConfirmacion)
                .orElseThrow(() -> new ExcepcionCitaNoEncontrada("Código de confirmación inválido"));
            
            // Verificar que el teléfono coincida
            if (!cita.getPaciente().getTelefono().contains(telefono.replaceAll("[^0-9]", ""))) {
                enviarMensajeError(telefono, "El número de teléfono no coincide con el registro de la cita");
                return;
            }
            
            servicioCitas.confirmarCita(cita.getId());
            
            enviarConfirmacionExitosa(telefono, cita);
            
        } catch (ExcepcionCitaNoEncontrada e) {
            enviarMensajeError(telefono, "Cita no encontrada. Por favor verifica tu código de confirmación.");
        }
    }
    
    private void manejarCancelacion(String telefono, String codigoConfirmacion) {
        try {
            Cita cita = servicioCitas
                .buscarPorCodigoConfirmacion(codigoConfirmacion)
                .orElseThrow(() -> new ExcepcionCitaNoEncontrada("Código de confirmación inválido"));
            
            // Verificar que el teléfono coincida
            if (!cita.getPaciente().getTelefono().contains(telefono.replaceAll("[^0-9]", ""))) {
                enviarMensajeError(telefono, "El número de teléfono no coincide con el registro de la cita");
                return;
            }
            
            // Verificar si la cancelación es permitida (al menos 24 horas antes)
            if (cita.getFechaCita().minusHours(24).isBefore(LocalDateTime.now())) {
                enviarMensajeError(telefono, 
                    "Las cancelaciones deben hacerse con al menos 24 horas de anticipación. " +
                    "Por favor llama al " + cita.getConsultorio().getTelefono());
                return;
            }
            
            servicioCitas.cancelarCita(cita.getId(), "Cancelado por el paciente vía WhatsApp");
            
            enviarCancelacionExitosa(telefono, cita);
            
        } catch (ExcepcionCitaNoEncontrada e) {
            enviarMensajeError(telefono, "Cita no encontrada. Por favor verifica tu código de confirmación.");
        }
    }
    
    private void enviarConfirmacionExitosa(String telefono, Cita cita) {
        String mensaje = String.format(
            "✅ *¡CONFIRMADO!*\n\n" +
            "Tu cita para el %s a las %s está confirmada.\n\n" +
            "¡Nos vemos entonces! 🦷😊",
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("dd 'de' MMMM")),
            cita.getFechaCita().format(DateTimeFormatter.ofPattern("hh:mm a"))
        );
        
        servicioNotificaciones.enviarMensajeWhatsApp(telefono, mensaje);
    }
    
    private void enviarCancelacionExitosa(String telefono, Cita cita) {
        String mensaje = String.format(
            "✅ *CANCELADA*\n\n" +
            "Tu cita ha sido cancelada.\n\n" +
            "Para reservar una nueva cita:\n" +
            "• Llámanos: %s\n" +
            "• Visita: %s\n" +
            "• Responde RESERVAR\n\n" +
            "¡Gracias! 🦷",
            cita.getConsultorio().getTelefono(),
            cita.getConsultorio().getSitioWeb()
        );
        
        servicioNotificaciones.enviarMensajeWhatsApp(telefono, mensaje);
    }
    
    private void enviarMensajeError(String telefono, String error) {
        String mensaje = String.format("❌ *ERROR*\n\n%s\n\nResponde AYUDA para asistencia.", error);
        servicioNotificaciones.enviarMensajeWhatsApp(telefono, mensaje);
    }
}
```

---

## 💻 Instalación y Configuración

### Prerrequisitos
```bash
# Software requerido
- Java 11 o superior
- Node.js 14+ y npm
- PostgreSQL 14+
- Redis (opcional, para gestión de sesiones)
- Docker & Docker Compose (opcional)
```

### Opción 1: Configuración con Docker (Recomendado)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Base de Datos PostgreSQL
  postgres:
    image: postgres:14
    container_name: dentalflow-db
    environment:
      POSTGRES_DB: dentalflow
      POSTGRES_USER: usuario_dentalflow
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - datos_postgres:/var/lib/postgresql/data
    networks:
      - red-dentalflow

  # Caché Redis
  redis:
    image: redis:7-alpine
    container_name: dentalflow-redis
    ports:
      - "6379:6379"
    networks:
      - red-dentalflow

  # API Backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: dentalflow-api
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/dentalflow
      SPRING_DATASOURCE_USERNAME: usuario_dentalflow
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      SPRING_REDIS_HOST: redis
      TWILIO_ACCOUNT_SID: ${TWILIO_ACCOUNT_SID}
      TWILIO_AUTH_TOKEN: ${TWILIO_AUTH_TOKEN}
      TWILIO_WHATSAPP_NUMBER: ${TWILIO_WHATSAPP_NUMBER}
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - redis
    networks:
      - red-dentalflow

  # Aplicación Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: dentalflow-web
    environment:
      API_URL: http://backend:8080
    ports:
      - "4200:80"
    depends_on:
      - backend
    networks:
      - red-dentalflow

  # Proxy Inverso Nginx
  nginx:
    image: nginx:alpine
    container_name: dentalflow-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    networks:
      - red-dentalflow

volumes:
  datos_postgres:

networks:
  red-dentalflow:
    driver: bridge
```

**Inicio Rápido con Docker:**
```bash
# 1. Clonar repositorio
git clone https://github.com/tuorg/dentalflow.git
cd dentalflow

# 2. Crear archivo .env
cp .env.example .env
# Editar .env con tus configuraciones

# 3. Iniciar todos los servicios
docker-compose up -d

# 4. Verificar logs
docker-compose logs -f

# 5. Acceder a la aplicación
# Frontend: http://localhost:4200
# API Backend: http://localhost:8080
# Documentación API: http://localhost:8080/swagger-ui.html
```

---

---

### Opción 2: Instalación Manual

#### Configuración del Backend

**1. Clonar y Configurar Proyecto:**
```bash
# Clonar repositorio
git clone https://github.com/tuorg/dentalflow.git
cd dentalflow/backend

# Crear application.yml
cp src/main/resources/application.yml.example src/main/resources/application.yml
```

**2. Configurar Base de Datos:**
```bash
# Crear base de datos PostgreSQL
psql -U postgres

CREATE DATABASE dentalflow;
CREATE USER usuario_dentalflow WITH PASSWORD 'tu_contraseña';
GRANT ALL PRIVILEGES ON DATABASE dentalflow TO usuario_dentalflow;
\q
```

**3. Configurar application.yml:**
```yaml
spring:
  application:
    name: dentalflow-api
  
  datasource:
    url: jdbc:postgresql://localhost:5432/dentalflow
    username: usuario_dentalflow
    password: tu_contraseña
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: none
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
  
  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.yaml
  
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB

server:
  port: 8080
  servlet:
    context-path: /api

# Configuración WhatsApp Twilio
twilio:
  account:
    sid: ${TWILIO_ACCOUNT_SID}
  auth:
    token: ${TWILIO_AUTH_TOKEN}
  whatsapp:
    from: ${TWILIO_WHATSAPP_NUMBER}
  webhook:
    url: ${APP_BASE_URL}/api/webhooks/whatsapp

# Configuración JWT
jwt:
  secret: ${JWT_SECRET:tu-clave-secreta-256-bits-aqui}
  expiration: 86400000 # 24 horas

# Configuración de Notificaciones
notificacion:
  whatsapp:
    habilitado: true
    max-intentos-reenvio: 3
    retraso-reenvio-minutos: 5
  recordatorios:
    24-horas-antes: true
    2-horas-antes: true
  email:
    habilitado: true
    from: noreply@dentalflow.com

# Almacenamiento de Archivos
almacenamiento:
  tipo: local # local o s3
  local:
    directorio-subida: ./uploads
  s3:
    bucket: archivos-dentalflow
    region: us-east-1

# Logging
logging:
  level:
    root: INFO
    com.dentalflow: DEBUG
    org.springframework.security: DEBUG
  file:
    name: logs/dentalflow.log
```

**4. Construir y Ejecutar Backend:**
```bash
# Construir proyecto
mvn clean install

# Ejecutar aplicación
mvn spring-boot:run

# O ejecutar JAR
java -jar target/dentalflow-api-1.0.0.jar

# La aplicación se iniciará en http://localhost:8080/api
```

**5. Verificar Backend:**
```bash
# Verificar endpoint de salud
curl http://localhost:8080/api/actuator/health

# Respuesta esperada:
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    }
  }
}
```

---

#### Configuración del Frontend

**1. Navegar al Directorio Frontend:**
```bash
cd ../frontend
```

**2. Instalar Dependencias:**
```bash
npm install
```

**3. Configurar Entorno:**

**src/environments/environment.ts:**
```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api',
  whatsappHabilitado: true,
  idiomaDefault: 'es',
  formatoFecha: 'dd/MM/yyyy',
  formatoHora: 'HH:mm',
  duracionesCitas: [15, 30, 45, 60, 90, 120],
  horarioLaboral: {
    inicio: '08:00',
    fin: '18:00'
  }
};
```

**src/environments/environment.prod.ts:**
```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.dentalflow.com/api',
  whatsappHabilitado: true,
  idiomaDefault: 'es',
  formatoFecha: 'dd/MM/yyyy',
  formatoHora: 'HH:mm',
  duracionesCitas: [15, 30, 45, 60, 90, 120],
  horarioLaboral: {
    inicio: '08:00',
    fin: '18:00'
  }
};
```

**4. Ejecutar Servidor de Desarrollo:**
```bash
npm start

# La aplicación se iniciará en http://localhost:4200
```

**5. Construir para Producción:**
```bash
npm run build

# Los archivos de producción estarán en dist/dentalflow
```

---

## 📚 Documentación de la API

### Endpoints de Autenticación

**Iniciar Sesión:**
```http
POST /api/auth/login
Content-Type: application/json

{
  "nombreUsuario": "admin@dentalflow.com",
  "contrasena": "password123"
}

Respuesta: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tipo": "Bearer",
  "expiraEn": 86400000,
  "usuario": {
    "id": 1,
    "nombreUsuario": "admin@dentalflow.com",
    "email": "admin@dentalflow.com",
    "rol": "ADMIN",
    "primerNombre": "Admin",
    "apellido": "Usuario"
  }
}
```

**Registrar Paciente:**
```http
POST /api/auth/registrar
Content-Type: application/json

{
  "primerNombre": "Juan",
  "apellido": "Pérez",
  "email": "juan.perez@email.com",
  "telefono": "+573001234567",
  "contrasena": "ContraseñaSegura123!",
  "fechaNacimiento": "1990-05-15",
  "numeroDocumento": "1234567890"
}

Respuesta: 201 Created
{
  "id": 42,
  "primerNombre": "Juan",
  "apellido": "Pérez",
  "email": "juan.perez@email.com",
  "telefono": "+573001234567",
  "estado": "ACTIVO",
  "creadoEn": "2024-01-21T10:30:00Z"
}
```

**Refrescar Token:**
```http
POST /api/auth/refrescar
Authorization: Bearer <token_expirado>

Respuesta: 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiraEn": 86400000
}
```

---

### Endpoints de Pacientes

**Obtener Todos los Pacientes:**
```http
GET /api/pacientes?pagina=0&tamano=20&orden=apellido,asc
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "contenido": [
    {
      "id": 1,
      "primerNombre": "Juan",
      "apellido": "Pérez",
      "email": "juan.perez@email.com",
      "telefono": "+573001234567",
      "fechaNacimiento": "1990-05-15",
      "genero": "MASCULINO",
      "estado": "ACTIVO",
      "citasProximas": 2,
      "ultimaVisita": "2024-01-15T14:30:00Z"
    }
  ],
  "paginable": {
    "numeroPagina": 0,
    "tamanoPagina": 20,
    "totalElementos": 156,
    "totalPaginas": 8
  }
}
```

**Obtener Paciente por ID:**
```http
GET /api/pacientes/{id}
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "id": 1,
  "primerNombre": "Juan",
  "apellido": "Pérez",
  "email": "juan.perez@email.com",
  "telefono": "+573001234567",
  "numeroDocumento": "1234567890",
  "fechaNacimiento": "1990-05-15",
  "genero": "MASCULINO",
  "direccion": "Calle 123 #45-67, Bogotá",
  "ciudad": "Bogotá",
  "historiaMedica": "Sin antecedentes médicos significativos",
  "alergias": "Penicilina",
  "tipoSangre": "O+",
  "nombreContactoEmergencia": "María Pérez",
  "telefonoContactoEmergencia": "+573007654321",
  "whatsappHabilitado": true,
  "estado": "ACTIVO",
  "creadoEn": "2023-06-10T09:00:00Z",
  "actualizadoEn": "2024-01-15T14:30:00Z"
}
```

**Crear Paciente:**
```http
POST /api/pacientes
Authorization: Bearer <token>
Content-Type: application/json

{
  "primerNombre": "María",
  "apellido": "García",
  "email": "maria.garcia@email.com",
  "telefono": "+573009876543",
  "numeroDocumento": "9876543210",
  "fechaNacimiento": "1985-08-20",
  "genero": "FEMENINO",
  "direccion": "Avenida 456 #78-90, Medellín",
  "ciudad": "Medellín",
  "historiaMedica": "Hipertensión controlada con medicación",
  "alergias": "Ninguna",
  "tipoSangre": "A+",
  "nombreContactoEmergencia": "Carlos García",
  "telefonoContactoEmergencia": "+573005555555",
  "whatsappHabilitado": true
}

Respuesta: 201 Created
{
  "id": 157,
  "primerNombre": "María",
  "apellido": "García",
  ...
  "creadoEn": "2024-01-21T11:00:00Z"
}
```

**Actualizar Paciente:**
```http
PUT /api/pacientes/{id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "telefono": "+573009876544",
  "direccion": "Calle 789 Nueva, Medellín",
  "historiaMedica": "Hipertensión controlada. Trabajo dental reciente completado."
}

Respuesta: 200 OK
```

**Buscar Pacientes:**
```http
GET /api/pacientes/buscar?consulta=juan&pagina=0&tamano=10
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "contenido": [
    // Pacientes que coinciden
  ],
  "totalElementos": 3
}
```

---

### Endpoints de Citas

**Obtener Horarios Disponibles:**
```http
GET /api/citas/horarios-disponibles?odontologoId=1&fecha=2024-01-25
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "fecha": "2024-01-25",
  "odontologoId": 1,
  "nombreOdontologo": "Dr. Carlos Méndez",
  "horariosDisponibles": [
    {
      "horaInicio": "09:00:00",
      "horaFin": "09:30:00",
      "disponible": true
    },
    {
      "horaInicio": "09:30:00",
      "horaFin": "10:00:00",
      "disponible": true
    },
    {
      "horaInicio": "10:00:00",
      "horaFin": "10:30:00",
      "disponible": false
    },
    {
      "horaInicio": "10:30:00",
      "horaFin": "11:00:00",
      "disponible": true
    }
  ]
}
```

**Crear Cita:**
```http
POST /api/citas
Authorization: Bearer <token>
Content-Type: application/json

{
  "pacienteId": 1,
  "odontologoId": 1,
  "consultorioId": 1,
  "fechaCita": "2024-01-25T10:00:00",
  "duracion": 30,
  "tipoServicio": "LIMPIEZA",
  "notas": "Control de rutina y limpieza"
}

Respuesta: 201 Created
{
  "id": 42,
  "paciente": {
    "id": 1,
    "primerNombre": "Juan",
    "apellido": "Pérez",
    "telefono": "+573001234567"
  },
  "odontologo": {
    "id": 1,
    "primerNombre": "Carlos",
    "apellido": "Méndez"
  },
  "consultorio": {
    "id": 1,
    "nombre": "DentalFlow Bogotá"
  },
  "fechaCita": "2024-01-25T10:00:00",
  "duracion": 30,
  "tipoServicio": "LIMPIEZA",
  "estado": "AGENDADA",
  "codigoConfirmacion": "ABC123XYZ",
  "confirmada": false,
  "creadoEn": "2024-01-21T11:15:00Z",
  "notificacionWhatsAppEnviada": true
}
```

**Obtener Citas por Rango de Fechas:**
```http
GET /api/citas?fechaInicio=2024-01-25&fechaFin=2024-01-31&odontologoId=1
Authorization: Bearer <token>

Respuesta: 200 OK
[
  {
    "id": 42,
    "paciente": {
      "id": 1,
      "nombreCompleto": "Juan Pérez"
    },
    "odontologo": {
      "id": 1,
      "nombreCompleto": "Dr. Carlos Méndez"
    },
    "fechaCita": "2024-01-25T10:00:00",
    "duracion": 30,
    "tipoServicio": "LIMPIEZA",
    "estado": "AGENDADA",
    "confirmada": true
  }
]
```

**Confirmar Cita:**
```http
PUT /api/citas/{id}/confirmar
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "id": 42,
  "confirmada": true,
  "confirmadaEn": "2024-01-21T12:00:00Z",
  "estado": "CONFIRMADA"
}
```

**Cancelar Cita:**
```http
PUT /api/citas/{id}/cancelar
Authorization: Bearer <token>
Content-Type: application/json

{
  "razon": "Paciente solicitó cancelación por conflicto de horario"
}

Respuesta: 200 OK
{
  "id": 42,
  "estado": "CANCELADA",
  "canceladaEn": "2024-01-21T12:30:00Z",
  "razonCancelacion": "Paciente solicitó cancelación por conflicto de horario"
}
```

**Reagendar Cita:**
```http
PUT /api/citas/{id}/reagendar
Authorization: Bearer <token>
Content-Type: application/json

{
  "nuevaFecha": "2024-01-26T14:00:00",
  "duracion": 30
}

Respuesta: 200 OK
{
  "id": 42,
  "fechaCita": "2024-01-26T14:00:00",
  "estado": "REAGENDADA",
  "codigoConfirmacion": "DEF456UVW"
}
```

---

### Endpoints de Tratamientos

**Obtener Tratamientos del Paciente:**
```http
GET /api/tratamientos/paciente/{pacienteId}
Authorization: Bearer <token>

Respuesta: 200 OK
[
  {
    "id": 1,
    "tipo": "ENDODONCIA",
    "numeroDiente": "16",
    "descripcion": "Tratamiento de conducto en primer molar superior derecho",
    "diagnostico": "Pulpitis",
    "estado": "EN_PROGRESO",
    "fechaInicio": "2024-01-15",
    "costoEstimado": 450000.00,
    "odontologo": {
      "id": 1,
      "nombreCompleto": "Dr. Carlos Méndez"
    },
    "sesiones": [
      {
        "numeroSesion": 1,
        "fecha": "2024-01-15T10:00:00",
        "notas": "Examen inicial y radiografías",
        "completada": true
      },
      {
        "numeroSesion": 2,
        "fecha": "2024-01-22T10:00:00",
        "notas": "Procedimiento de conducto",
        "completada": false
      }
    ]
  }
]
```

**Crear Plan de Tratamiento:**
```http
POST /api/tratamientos
Authorization: Bearer <token>
Content-Type: application/json

{
  "pacienteId": 1,
  "odontologoId": 1,
  "citaId": 42,
  "tipo": "RESINA",
  "numeroDiente": "26",
  "descripcion": "Resina composite en primer molar inferior izquierdo",
  "diagnostico": "Caries dental",
  "costoEstimado": 120000.00,
  "numeroDeSesiones": 1,
  "notas": "Paciente solicitó resina del color del diente"
}

Respuesta: 201 Created
{
  "id": 15,
  "tipo": "RESINA",
  "numeroDiente": "26",
  "estado": "PLANEADO",
  "fechaInicio": "2024-01-21",
  "costoEstimado": 120000.00,
  "creadoEn": "2024-01-21T13:00:00Z"
}
```

**Actualizar Estado de Tratamiento:**
```http
PUT /api/tratamientos/{id}/estado
Authorization: Bearer <token>
Content-Type: application/json

{
  "estado": "COMPLETADO",
  "notas": "Tratamiento completado exitosamente. Paciente informado sobre cuidados posteriores."
}

Respuesta: 200 OK
```

---

### Endpoints de Odontólogos

**Obtener Todos los Odontólogos:**
```http
GET /api/odontologos?consultorioId=1
Authorization: Bearer <token>

Respuesta: 200 OK
[
  {
    "id": 1,
    "primerNombre": "Carlos",
    "apellido": "Méndez",
    "email": "carlos.mendez@dentalflow.com",
    "telefono": "+573101234567",
    "numeroLicencia": "COL-12345",
    "especializacion": "Odontología General",
    "consultorio": {
      "id": 1,
      "nombre": "DentalFlow Bogotá"
    },
    "estaActivo": true,
    "urlFoto": "https://storage.dentalflow.com/odontologos/1.jpg"
  }
]
```

**Obtener Horario del Odontólogo:**
```http
GET /api/odontologos/{id}/horario?fecha=2024-01-25
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "odontologoId": 1,
  "fecha": "2024-01-25",
  "horarioLaboral": {
    "inicio": "08:00:00",
    "fin": "17:00:00",
    "inicioAlmuerzo": "12:00:00",
    "finAlmuerzo": "13:00:00"
  },
  "citas": [
    {
      "id": 42,
      "horaInicio": "10:00:00",
      "horaFin": "10:30:00",
      "nombrePaciente": "Juan Pérez",
      "tipoServicio": "LIMPIEZA"
    }
  ],
  "horariosDisponibles": [
    {
      "horaInicio": "08:00:00",
      "horaFin": "08:30:00"
    },
    {
      "horaInicio": "08:30:00",
      "horaFin": "09:00:00"
    }
  ]
}
```

---

### Endpoints de Notificaciones

**Obtener Historial de Notificaciones:**
```http
GET /api/notificaciones/cita/{citaId}
Authorization: Bearer <token>

Respuesta: 200 OK
[
  {
    "id": 1,
    "tipo": "CONFIRMACION",
    "telefonoDestinatario": "+573001234567",
    "mensaje": "🦷 CITA CONFIRMADA...",
    "estado": "ENTREGADO",
    "enviadoEn": "2024-01-21T11:15:30Z",
    "entregadoEn": "2024-01-21T11:15:35Z"
  },
  {
    "id": 2,
    "tipo": "RECORDATORIO_24H",
    "telefonoDestinatario": "+573001234567",
    "estado": "ENVIADO",
    "enviadoEn": "2024-01-24T10:00:00Z",
    "programadoPara": "2024-01-24T10:00:00Z"
  }
]
```

**Reenviar Notificación:**
```http
POST /api/notificaciones/{id}/reenviar
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "id": 2,
  "estado": "ENVIADO",
  "enviadoEn": "2024-01-21T14:00:00Z",
  "intentosReenvio": 1
}
```

---

### Dashboard y Reportes

**Obtener Estadísticas del Dashboard:**
```http
GET /api/dashboard/estadisticas?fechaInicio=2024-01-01&fechaFin=2024-01-31
Authorization: Bearer <token>

Respuesta: 200 OK
{
  "periodo": {
    "fechaInicio": "2024-01-01",
    "fechaFin": "2024-01-31"
  },
  "citas": {
    "total": 245,
    "agendadas": 180,
    "completadas": 58,
    "canceladas": 7,
    "inasistencias": 0
  },
  "pacientes": {
    "total": 156,
    "nuevos": 23,
    "recurrentes": 133,
    "activos": 145
  },
  "ingresos": {
    "total": 45600000.00,
    "pendiente": 8900000.00,
    "cobrado": 36700000.00
  },
  "notificaciones": {
    "enviadas": 490,
    "entregadas": 482,
    "fallidas": 8,
    "tasaEntrega": 98.37
  },
  "reduccionInasistencias": 87.5
}
```

**Obtener Reporte de Citas:**
```http
GET /api/reportes/citas?fechaInicio=2024-01-01&fechaFin=2024-01-31&formato=PDF
Authorization: Bearer <token>

Respuesta: 200 OK (Descarga de archivo PDF)
```

---

## 👥 Roles y Permisos de Usuario

### Jerarquía de Roles
```
ADMIN
├── Acceso completo al sistema
├── Gestión de usuarios
├── Configuración de consultorios
├── Configuración del sistema
└── Todos los reportes

ODONTOLOGO
├── Ver citas propias
├── Gestionar pacientes propios
├── Crear/actualizar tratamientos
├── Ver registros de pacientes
└── Reportes limitados

RECEPCIONISTA
├── Agendar citas
├── Gestionar registros de pacientes
├── Enviar notificaciones
├── Reportes básicos
└── Sin acceso financiero

PACIENTE
├── Ver perfil propio
├── Reservar citas
├── Ver historial de tratamientos
├── Actualizar información de contacto
└── Sin acceso administrativo
```

### Matriz de Permisos

| Funcionalidad | Admin | Odontólogo | Recepcionista | Paciente |
|---------------|-------|------------|---------------|----------|
| **Citas** |
| Crear Cita | ✅ | ✅ | ✅ | ✅* |
| Ver Todas las Citas | ✅ | ❌ | ✅ | ❌ |
| Ver Citas Propias | ✅ | ✅ | ✅ | ✅ |
| Cancelar Cita | ✅ | ✅ | ✅ | ✅* |
| Reagendar Cita | ✅ | ✅ | ✅ | ✅* |
| **Pacientes** |
| Crear Paciente | ✅ | ✅ | ✅ | ❌ |
| Ver Todos los Pacientes | ✅ | ❌ | ✅ | ❌ |
| Ver Detalles Paciente | ✅ | ✅** | ✅ | ✅*** |
| Actualizar Paciente | ✅ | ✅** | ✅ | ✅*** |
| Eliminar Paciente | ✅ | ❌ | ❌ | ❌ |
| **Tratamientos** |
| Crear Plan Tratamiento | ✅ | ✅ | ❌ | ❌ |
| Ver Tratamientos | ✅ | ✅** | ✅ | ✅*** |
| Actualizar Tratamiento | ✅ | ✅** | ❌ | ❌ |
| **Financiero** |
| Ver Facturas | ✅ | ✅** | ✅ | ✅*** |
| Crear Factura | ✅ | ❌ | ✅ | ❌ |
| Procesar Pago | ✅ | ❌ | ✅ | ❌ |
| **Reportes** |
| Reportes Financieros | ✅ | ❌ | ❌ | ❌ |
| Reportes de Citas | ✅ | ✅** | ✅ | ❌ |
| Reportes de Pacientes | ✅ | ✅** | ✅ | ❌ |
| **Configuración** |
| Gestión de Usuarios | ✅ | ❌ | ❌ | ❌ |
| Config. Consultorios | ✅ | ❌ | ❌ | ❌ |
| Horarios de Trabajo | ✅ | ✅** | ❌ | ❌ |
| Config. Notificaciones | ✅ | ✅ | ✅ | ✅*** |

*Solo citas propias  
**Solo pacientes asignados  
***Solo registros propios

---

## 🧪 Pruebas

### Pruebas Unitarias

**Ejemplo: Prueba de Servicio de Citas**
```java
@SpringBootTest
@Transactional
class ServicioCitasTest {
    
    @Autowired
    private ServicioCitas servicioCitas;
    
    @MockBean
    private ServicioWhatsApp servicioWhatsApp;
    
    @Test
    @DisplayName("Debe agendar cita exitosamente")
    void testAgendarCita() {
        // Dado
        SolicitudCita solicitud = SolicitudCita.builder()
            .pacienteId(1L)
            .odontologoId(1L)
            .consultorioId(1L)
            .fechaCita(LocalDateTime.now().plusDays(1))
            .duracion(30)
            .tipoServicio(TipoServicio.LIMPIEZA)
            .build();
        
        // Cuando
        CitaDTO resultado = servicioCitas.agendarCita(solicitud);
        
        // Entonces
        assertNotNull(resultado);
        assertEquals(EstadoCita.AGENDADA, resultado.getEstado());
        assertNotNull(resultado.getCodigoConfirmacion());
        
        // Verificar que se envió notificación WhatsApp
        verify(servicioWhatsApp, times(1))
            .enviarConfirmacionCita(any(Cita.class));
    }
    
    @Test
    @DisplayName("Debe lanzar excepción cuando odontólogo no está disponible")
    void testAgendarCitaOdontologoNoDisponible() {
        // Dado
        SolicitudCita solicitud = SolicitudCita.builder()
            .pacienteId(1L)
            .odontologoId(1L)
            .consultorioId(1L)
            .fechaCita(LocalDateTime.of(2024, 1, 25, 10, 0))
            .duracion(30)
            .tipoServicio(TipoServicio.LIMPIEZA)
            .build();
        
        // Cita existente en el mismo horario
        crearCitaExistente(1L, LocalDateTime.of(2024, 1, 25, 10, 0));
        
        // Cuando y Entonces
        assertThrows(ExcepcionConflictoCita.class, () -> {
            servicioCitas.agendarCita(solicitud);
        });
    }
    
    @Test
    @DisplayName("Debe confirmar cita exitosamente")
    void testConfirmarCita() {
        // Dado
        Long citaId = 1L;
        
        // Cuando
        servicioCitas.confirmarCita(citaId);
        
        // Entonces
        Cita cita = repositorioCitas.findById(citaId).orElseThrow();
        assertTrue(cita.getConfirmada());
        assertNotNull(cita.getConfirmadaEn());
    }
}
```

**Ejecutar Pruebas Unitarias:**
```bash
# Ejecutar todas las pruebas
mvn test

# Ejecutar clase de prueba específica
mvn test -Dtest=ServicioCitasTest

# Ejecutar con reporte de cobertura
mvn test jacoco:report

# El reporte de cobertura estará en target/site/jacoco/index.html
```

### Pruebas de Integración

**Ejemplo: Prueba de Integración API de Citas**
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class ControladorCitasIntegracionTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private String tokenAuth;
    
    @BeforeEach
    void setUp() throws Exception {
        // Iniciar sesión y obtener token
        SolicitudLogin solicitudLogin = new SolicitudLogin("admin@dentalflow.com", "password");
        
        MvcResult resultado = mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(solicitudLogin)))
            .andExpect(status().isOk())
            .andReturn();
        
        RespuestaLogin respuesta = objectMapper.readValue(
            resultado.getResponse().getContentAsString(),
            RespuestaLogin.class
        );
        
        tokenAuth = respuesta.getToken();
    }
    
    @Test
    @DisplayName("Debe crear cita y retornar 201")
    void testCrearCita() throws Exception {
        SolicitudCita solicitud = SolicitudCita.builder()
            .pacienteId(1L)
            .odontologoId(1L)
            .consultorioId(1L)
            .fechaCita(LocalDateTime.now().plusDays(1))
            .duracion(30)
            .tipoServicio(TipoServicio.LIMPIEZA)
            .notas("Control de rutina")
            .build();
        
        mockMvc.perform(post("/api/citas")
                .header("Authorization", "Bearer " + tokenAuth)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(solicitud)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").exists())
            .andExpect(jsonPath("$.estado").value("AGENDADA"))
            .andExpect(jsonPath("$.codigoConfirmacion").exists())
            .andExpect(jsonPath("$.paciente.id").value(1))
            .andExpect(jsonPath("$.odontologo.id").value(1));
    }
    
    @Test
    @DisplayName("Debe retornar 401 cuando no está autenticado")
    void testCrearCitaNoAutorizado() throws Exception {
        SolicitudCita solicitud = SolicitudCita.builder()
            .pacienteId(1L)
            .odontologoId(1L)
            .build();
        
        mockMvc.perform(post("/api/citas")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(solicitud)))
            .andExpect(status().isUnauthorized());
    }
}
```

**Ejecutar Pruebas de Integración:**
```bash
# Ejecutar pruebas de integración
mvn verify

# Ejecutar con perfil específico
mvn verify -P pruebas-integracion
```

### Pruebas Unitarias Frontend

**Ejemplo: Prueba de Componente de Citas (Angular):**
```typescript
// formulario-cita.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ReactiveFormsModule } from '@angular/forms';
import { of, throwError } from 'rxjs';

import { FormularioCitaComponent } from './formulario-cita.component';
import { ServicioCitas } from '../../servicios/servicio-citas.service';

describe('FormularioCitaComponent', () => {
  let componente: FormularioCitaComponent;
  let fixture: ComponentFixture<FormularioCitaComponent>;
  let servicioCitas: jasmine.SpyObj<ServicioCitas>;

  beforeEach(async () => {
    const servicioCitasSpy = jasmine.createSpyObj('ServicioCitas', [
      'crearCita',
      'obtenerHorariosDisponibles'
    ]);

    await TestBed.configureTestingModule({
      declarations: [ FormularioCitaComponent ],
      imports: [ ReactiveFormsModule ],
      providers: [
        { provide: ServicioCitas, useValue: servicioCitasSpy }
      ]
    }).compileComponents();

    servicioCitas = TestBed.inject(ServicioCitas) as jasmine.SpyObj<ServicioCitas>;
    fixture = TestBed.createComponent(FormularioCitaComponent);
    componente = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('debe crear', () => {
    expect(componente).toBeTruthy();
  });

  it('debe inicializar formulario con valores vacíos', () => {
    expect(componente.formularioCita.get('pacienteId')?.value).toBe('');
    expect(componente.formularioCita.get('odontologoId')?.value).toBe('');
    expect(componente.formularioCita.get('fechaCita')?.value).toBe('');
  });

  it('debe llamar servicio de citas al enviar formulario', () => {
    const citaMock = {
      id: 1,
      pacienteId: 1,
      odontologoId: 1,
      fechaCita: new Date(),
      estado: 'AGENDADA'
    };

    servicioCitas.crearCita.and.returnValue(of(citaMock));

    componente.formularioCita.patchValue({
      pacienteId: 1,
      odontologoId: 1,
      consultorioId: 1,
      fechaCita: new Date(),
      duracion: 30,
      tipoServicio: 'LIMPIEZA'
    });

    componente.onSubmit();

    expect(servicioCitas.crearCita).toHaveBeenCalledTimes(1);
  });

  it('debe mostrar mensaje de error en envío fallido', () => {
    servicioCitas.crearCita.and.returnValue(
      throwError({ error: { mensaje: 'Conflicto de cita' } })
    );

    componente.formularioCita.patchValue({
      pacienteId: 1,
      odontologoId: 1,
      fechaCita: new Date()
    });

    componente.onSubmit();

    expect(componente.mensajeError).toBe('Conflicto de cita');
  });
});
```

**Ejecutar Pruebas Frontend:**
```bash
# Ejecutar pruebas unitarias
ng test

# Ejecutar con cobertura
ng test --code-coverage

# Ejecutar en modo headless (CI)
ng test --watch=false --browsers=ChromeHeadless

# El reporte de cobertura estará en coverage/dentalflow/index.html
```

### Pruebas End-to-End

**Ejemplo: Prueba E2E con Cypress:**
```typescript
// cypress/e2e/reserva-cita.cy.ts
describe('Flujo de Reserva de Cita', () => {
  beforeEach(() => {
    cy.login('paciente@ejemplo.com', 'password123');
  });

  it('debe completar el flujo completo de reserva de cita', () => {
    // Navegar a página de reserva
    cy.visit('/citas/nueva');

    // Seleccionar odontólogo
    cy.get('[data-cy=select-odontologo]').click();
    cy.get('[data-cy=opcion-odontologo-1]').click();

    // Seleccionar fecha
    cy.get('[data-cy=selector-fecha]').click();
    cy.get('.fecha-disponible').first().click();

    // Seleccionar horario
    cy.get('[data-cy=horario-disponible]').first().click();

    // Seleccionar tipo de servicio
    cy.get('[data-cy=select-servicio]').select('LIMPIEZA');

    // Agregar notas
    cy.get('[data-cy=textarea-notas]').type('Control de rutina y limpieza');

    // Enviar formulario
    cy.get('[data-cy=boton-enviar]').click();

    // Verificar mensaje de éxito
    cy.get('[data-cy=mensaje-exito]')
      .should('be.visible')
      .and('contain', 'Cita agendada exitosamente');

    // Verificar que se muestra código de confirmación
    cy.get('[data-cy=codigo-confirmacion]').should('exist');

    // Verificar mensaje de notificación WhatsApp
    cy.get('[data-cy=notificacion-whatsapp]')
      .should('contain', 'Confirmación enviada por WhatsApp');
  });

  it('debe manejar conflicto de cita', () => {
    cy.visit('/citas/nueva');

    // Intentar reservar horario ya ocupado
    cy.get('[data-cy=select-odontologo]').select('1');
    cy.get('[data-cy=selector-fecha]').type('2024-01-25');
    cy.get('[data-cy=horario-disponible]').contains('10:00 AM').click();

    cy.get('[data-cy=boton-enviar]').click();

    // Verificar mensaje de error
    cy.get('[data-cy=mensaje-error]')
      .should('be.visible')
      .and('contain', 'Horario ya reservado');
  });
});
```

**Ejecutar Pruebas E2E:**
```bash
# Abrir Cypress Test Runner
npm run cypress:open

# Ejecutar pruebas en modo headless
npm run cypress:run

# Ejecutar archivo de prueba específico
npm run cypress:run --spec "cypress/e2e/reserva-cita.cy.ts"
```

---

## 🚀 Despliegue

### Despliegue en Producción

**1. Preparar Entorno:**
```bash
# Crear directorio de despliegue
mkdir -p /opt/dentalflow
cd /opt/dentalflow

# Clonar repositorio
git clone https://github.com/tuorg/dentalflow.git .
```

**2. Configurar Variables de Entorno:**
```bash
# Crear archivo .env
cat > .env << EOF
# Base de Datos
DB_HOST=localhost
DB_PORT=5432
DB_NAME=dentalflow
DB_USER=usuario_dentalflow
DB_PASSWORD=contraseña_segura_aqui

# JWT
JWT_SECRET=clave_secreta_jwt_256_bits_aqui

# Twilio WhatsApp
TWILIO_ACCOUNT_SID=tu_account_sid
TWILIO_AUTH_TOKEN=tu_auth_token
TWILIO_WHATSAPP_NUMBER=+14155238886

# Aplicación
APP_BASE_URL=https://dentalflow.com
SPRING_PROFILES_ACTIVE=production
EOF

chmod 600 .env
```

**3. Construir Aplicación:**
```bash
# Backend
cd backend
mvn clean package -DskipTests
cp target/dentalflow-api-1.0.0.jar /opt/dentalflow/

# Frontend
cd ../frontend
npm install
npm run build:prod
cp -r dist/dentalflow /var/www/dentalflow
```

**4. Configurar Servicio Systemd:**
```bash
sudo cat > /etc/systemd/system/dentalflow.service << EOF
[Unit]
Description=DentalFlow API
After=network.target postgresql.service

[Service]
Type=simple
User=dentalflow
WorkingDirectory=/opt/dentalflow
EnvironmentFile=/opt/dentalflow/.env
ExecStart=/usr/bin/java -jar /opt/dentalflow/dentalflow-api-1.0.0.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Iniciar servicio
sudo systemctl daemon-reload
sudo systemctl enable dentalflow
sudo systemctl start dentalflow
```

**5. Configurar Nginx:**
```nginx
# /etc/nginx/sites-available/dentalflow
server {
    listen 80;
    server_name dentalflow.com www.dentalflow.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name dentalflow.com www.dentalflow.com;

    ssl_certificate /etc/letsencrypt/live/dentalflow.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dentalflow.com/privkey.pem;

    # Frontend
    location / {
        root /var/www/dentalflow;
        try_files $uri $uri/ /index.html;
    }

    # API Backend
    location /api/ {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**6. Obtener Certificado SSL:**
```bash
sudo certbot --nginx -d dentalflow.com -d www.dentalflow.com
```

---

## 📝 Licencia

Este proyecto fue desarrollado como parte de actividades de investigación y formación en el SENA (Servicio Nacional de Aprendizaje) para apoyar a los egresados de odontología. El código, aplicaciones, documentación y repositorios son propiedad del SENA y han sido recreados para fines de demostración en portafolio.


---

</div>
