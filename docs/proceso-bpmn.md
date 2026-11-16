# Proceso BPMN: asignación de recursos técnicos

**Pool cliente:** solicitante del proyecto (fuera del control interno). **Pool CodeNova:** lanes Gestión de Proyectos, SGPI e Infraestructura y Recursos. El diagrama muestra la solicitud, validación, decisión por disponibilidad y confirmación. La reserva se persiste antes de confirmar.

```mermaid
flowchart LR
  subgraph C[Pool externo: Cliente o solicitante]
    C1((Inicio: necesidad)) --> C2[Enviar solicitud con proyecto periodo y cantidad]
    C3[Recibir resultado]
  end
  subgraph N[Pool interno: CodeNova]
    subgraph G[Lane: Gestión de Proyectos]
      G1[Registrar solicitud en SGPI]
      G2[Corregir datos o cambiar periodo o recurso]
      G3[Comunicar confirmación o rechazo]
    end
    subgraph S[Lane: SGPI]
      S1[Validar usuario proyecto permisos y datos]
      D1{¿Solicitud válida?}
      S2[Consultar estado vencimiento y reservas coincidentes]
      D2{¿Recurso disponible y apto?}
      S3[Guardar asignación de forma atómica y registrar trazabilidad]
      S4[Informar error y no guardar reserva]
    end
    subgraph I[Lane: Infraestructura y Recursos]
      I1[Verificar inventario y atender excepciones de renovación]
    end
    G1 --> S1 --> D1
    D1 -- No --> S4 --> G2 --> S1
    D1 -- Sí --> S2 --> D2
    D2 -- Sí --> S3 --> G3
    D2 -- No --> I1 --> G2
    G3 --> G3
  end
  C2 --> G1
  G3 --> C3
  C3 --> C4((Fin))
```

## Reglas y excepciones

- La persona solicitante debe estar activa y autorizada; proyecto y recurso deben existir.
- No se asignan equipos en mantenimiento o baja. En licencias con vencimiento, la vigencia debe cubrir todo el período.
- La suma de asignaciones coincidentes no supera `capacidad_uso`. La verificación y escritura se realizan como una operación atómica para prevenir reservas simultáneas incompatibles.
- Si los datos son inválidos, se informa el motivo y Gestión corrige la solicitud. Si no hay disponibilidad, puede cambiar período o recurso y revalidar.
- Una licencia vencida se deriva a Infraestructura para renovación por su circuito; la solicitud se vuelve a validar después.
- Solo se confirma luego de guardar. Si falla el guardado, no se envía confirmación.

**Nota:** diagrama de diseño para el caso académico; no describe un proceso productivo ya implementado.
