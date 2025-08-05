# 🧠 Uso de Spring Data JPA en `Principal.java`

Este documento resume cómo se integra JPA en la clase `Principal` para gestionar la persistencia de objetos `Serie` y `Episodio` en el proyecto **Screenmatch**.

---

## 📦 Repositorio y método `findAll()`

El repositorio está inyectado en el constructor:

```java
private SerieRepository repositorio;

public Principal(SerieRepository repository) {
    this.repositorio = repository;
}
```

Cuando se ejecuta:

```java
series = repositorio.findAll();
```

Spring Data JPA realiza internamente:

```sql
SELECT * FROM series;
```

Y devuelve una `List<Serie>`, donde cada objeto `Serie` representa una fila en la tabla.

## 🎞️ Llenado manual de episodios desde la API

El método `buscarEpisodioPorSerie()` permite llenar episodios de una serie existente:

```java
Optional<Serie> serie = series.stream()
    .filter(s -> s.getTitulo() != null && s.getTitulo().toLowerCase().contains(nombreSerie))
    .findFirst();
```

Por cada temporada de la serie encontrada, se hace una llamada a la API OMDB:

```java
for (int i = 1; i <= serieEncontrada.getTotalTemporadas(); i++) {
    var json = consumoApi.obtenerDatos(URL_BASE + serieEncontrada.getTitulo().replace(" ", "+") + "&season=" + i + API_KEY);
    DatosTemporadas datosTemporada = conversor.obtenerDatos(json, DatosTemporadas.class);
    temporadas.add(datosTemporada);
}
```

Después, se transforman los datos de la API en objetos `Episodio`:

```java
List<Episodio> episodios = temporadas.stream()
    .flatMap(d -> d.episodios().stream()
        .map(e -> new Episodio(d.numero(), e)))
    .collect(Collectors.toList());

serieEncontrada.setEpisodios(episodios);
repositorio.save(serieEncontrada);
```

## 📋 Visualización de series ordenadas por evaluación

Cuando el usuario elige la opción 3 del menú:

```java
series = repositorio.findAll();
series.stream()
    .filter(s -> s.getEvaluacion() != null)
    .sorted(Comparator.comparing(Serie::getEvaluacion))
    .forEach(System.out::println);
```

Esto imprime las series almacenadas, ordenadas por evaluación.

## 🔗 Relación Serie ↔ Episodio

```java
// En Serie.java
@OneToMany(mappedBy = "serie", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
private List<Episodio> episodios;

// En Episodio.java
@ManyToOne
private Serie serie;
```

- **@OneToMany**: una serie puede tener muchos episodios
- **cascade = ALL**: al guardar la serie, también se guardan sus episodios
- **fetch = EAGER**: al consultar la serie, se cargan los episodios automáticamente

## ✅ Conclusión

- La clase `Principal` actúa como controlador de flujo
- El repositorio permite guardar y recuperar objetos `Serie`
- Los episodios se cargan desde la API y se asocian manualmente
- Al persistirlos, se mantienen las relaciones gracias a JPA

Este archivo sirve como guía para comprender cómo se hace persistencia y recuperación de datos usando Spring Boot + JPA en el proyecto **screenmatch**.