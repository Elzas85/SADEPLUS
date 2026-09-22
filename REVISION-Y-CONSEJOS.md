# SADE+ - revision y consejos

Fecha: 12 de septiembre de 2026

## Estado general

SADE+ esta bien orientado para una herramienta interna de lectura y organizacion:

- replica el listado sin modificar el sistema original;
- muestra texto completo, busqueda, orden y paginacion;
- mantiene etiquetas y anotaciones separadas del expediente;
- verifica que la accion disponible diga exactamente `Tramitar`;
- permite abrir la actuacion en una pestana nueva;
- incluye exportacion e importacion de anotaciones;
- funciona tanto sobre Expediente Electronico como sobre GEDO.

Como SADE+ es para uso interno, el objetivo no es publicarlo para todo el mundo. El objetivo es que sea confiable para vos y tus colegas y que no pierda anotaciones ni dispare una accion dos veces.

## 1. Evitar que dos pestañas se pisen las anotaciones

### Problema

SADE+ carga todas las marcas en la variable `MARCAS`. Cuando se modifica una fila, `fijarMarca` cambia esa copia en memoria y guarda el objeto completo.

Si hay dos pestañas abiertas:

1. la pestaña A lee las marcas;
2. la pestaña B lee las mismas marcas;
3. A agrega una anotacion y guarda;
4. B agrega otra anotacion usando su copia vieja y guarda;
5. el guardado de B puede borrar el cambio de A.

### Por que importa

Es una perdida silenciosa de trabajo. Puede ocurrir justo cuando se trabaja con un expediente en una pestaña y el listado en otra.

### Consejo

Antes de cada cambio:

1. volver a leer el objeto guardado;
2. aplicar solamente el cambio de la fila que se esta editando;
3. guardar el resultado actualizado.

Para hacerlo mas robusto, agregar una marca de tiempo por fila. Si dos copias modifican filas distintas, se pueden combinar. Si modifican la misma fila, se puede conservar el texto mas nuevo o conservar ambos con una separacion visible.

Tambien conviene escuchar el evento `storage` cuando se usa `localStorage`, y redibujar el listado si otra pestaña modifico una anotacion.

### Prueba necesaria

1. Abrir SADE+ en dos pestañas.
2. Anotar una actuacion en la pestaña A.
3. Anotar otra actuacion en la pestaña B.
4. Recargar ambas.
5. Confirmar que sobreviven las dos anotaciones.
6. Repetir modificando la misma fila y comprobar que no se pierde texto sin aviso.

## 2. Hacer que el encargo de abrir una actuación sea propio de una ventana

### Problema

Para abrir una actuación en otra pestaña, SADE+ deja un encargo en una unica clave de `localStorage`:

```js
sade.plus.abrir
```

La pestaña nueva lo toma y lo borra. Si hay otra pestaña de SADE+ cargando al mismo tiempo, puede leer ese encargo antes que la pestaña correcta.

### Que puede pasar

La actuacion podria tramitarse en la pestaña equivocada, o el encargo podria desaparecer si la pestaña nueva tarda demasiado en cargar.

### Consejo

Agregar un identificador aleatorio de encargo y asociarlo a la ventana que se abre:

- `id` aleatorio;
- fecha de creacion;
- clave de actuacion;
- origen esperado;
- estado `pendiente`.

La pestaña nueva deberia reclamar el encargo usando ese identificador. Si no puede reclamarlo, no debe ejecutar `Tramitar`.

Como minimo, incluir la URL esperada y no permitir que una pantalla GEDO consuma un encargo de Expediente Electronico.

### Pruebas necesarias

- Abrir dos actuaciones rapidamente.
- Tener dos pestañas de SADE+ abiertas antes de abrir una nueva.
- Abrir una actuacion con GEDO visible.
- Bloquear la ventana emergente y confirmar que no queda un encargo pendiente.
- Recargar la pestaña nueva antes de que aparezca el listado.

## 3. Proteger mejor la accion Tramitar

La comprobacion de que la opcion diga exactamente `Tramitar` es una buena proteccion y debe conservarse.

Como refuerzo:

- deshabilitar temporalmente el boton de abrir mientras se espera la respuesta;
- guardar la clave de la actuacion en curso;
- rechazar otro intento con la misma clave;
- despues de hacer clic, comprobar que la fila cambio o que la pantalla de expediente aparecio;
- si no cambia, informar `No se pudo confirmar que Tramitar haya sido ejecutado`.

No conviene reintentar automaticamente `Tramitar`: es una orden sobre el sistema y podria ejecutarse dos veces.

## 4. Respaldos y uso interno

Como SADE+ es para vos y tus colegas:

- mantener el repositorio privado;
- no incluir expedientes, anotaciones ni exportaciones reales;
- no guardar datos sensibles en capturas de prueba;
- mantener una carpeta de ejemplos anonimizados;
- explicar al equipo que las anotaciones viven en el navegador de cada computadora;
- recomendar exportarlas como respaldo antes de limpiar el perfil del navegador.

Si varias personas usan la misma computadora, conviene que cada perfil del navegador tenga su propio almacenamiento. Si no es posible, las anotaciones necesitan una separacion adicional por usuario.

## 5. Mejoras recomendadas despues

### Registro de fallas

Cuando `Tramitar`, la lectura o la apertura de una ficha fallan, guardar localmente un registro breve:

- fecha y hora;
- modulo: Expediente o GEDO;
- etapa;
- mensaje;
- clave de actuacion anonimizada.

No guardar el contenido completo del expediente.

### Estado de carga

En listados grandes conviene indicar:

- pagina actual;
- cantidad de filas leidas;
- si la ventana muestra todo el listado o solo lo que ya llego;
- si una anotacion se guardo correctamente.

### Consistencia de vocabulario

Usar siempre:

- `anotacion` para datos privados;
- `Tramitar` para la accion del sistema;
- `abrir` para mostrar una pantalla;
- `guardar` para persistir una anotacion;
- `exportar` e `importar` para mover datos entre computadoras.

Eso evita confundir una anotacion interna con una accion del expediente.

## Orden sugerido

1. Evitar que dos pestañas pisen anotaciones.
2. Aislar y validar los encargos de pestanas nuevas.
3. Bloquear acciones duplicadas de `Tramitar`.
4. Agregar pruebas de dos pestañas y recargas.
5. Mejorar el registro de fallas y los textos visibles.

## Verificacion actual

El userscript pasa la validacion de sintaxis y el editor no informa errores. Esta revision es estatica. Las pruebas reales deben hacerse primero sobre una actuacion que no implique una accion sensible y, para `Tramitar`, solo cuando el usuario este seguro de que corresponde ejecutarlo.
