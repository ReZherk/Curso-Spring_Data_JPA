# 🔗 Explicación de Anotaciones JPA en las Entidades `Serie` y `Episodio`

Este documento resume el uso de las principales anotaciones de JPA en el proyecto **Screenmatch**, con un enfoque en la relación entre `Serie` y `Episodio`.

---

## 🧩 @OneToMany y @ManyToOne

### En `Serie.java`:

```java
@OneToMany(mappedBy = "serie", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
private List<Episodio> episodios;
```

- **@OneToMany**: Una serie puede tener muchos episodios
- **mappedBy = "serie"**: Indica que la relación está controlada desde el atributo `serie` en `Episodio`
- **cascade = CascadeType.ALL**: Guardar, actualizar o eliminar una serie también afecta a sus episodios
- **fetch = FetchType.EAGER**: Al cargar una serie, también se cargan automáticamente todos sus episodios desde la base de datos

👉 **Por eso no se crea ninguna columna para episodios en la tabla `series`. La relación está definida del lado de episodios.**

### En `Episodio.java`:

```java
@ManyToOne
private Serie serie;
```

- **@ManyToOne**: Muchos episodios pueden pertenecer a una misma serie
- Esto crea una columna `serie_id` en la tabla `episodios`, que actúa como clave foránea

## 🧠 ¿Qué hace fetch = FetchType.EAGER?

Cuando se consulta una serie:

```java
Serie s = repositorio.findById(1L).get();
```

JPA ejecuta automáticamente:

```sql
SELECT * FROM series WHERE id = 1;
SELECT * FROM episodios WHERE serie_id = 1;
```

→ Así, `s.getEpisodios()` ya viene cargado sin necesidad de otra consulta manual.

⚠️ **fetch = EAGER no guarda datos, solo afecta cómo se recuperan los datos ya guardados en la base.**

## 🏷️ @GeneratedValue y Clave Primaria

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

- **@Id**: Marca este campo como clave primaria
- **@GeneratedValue(...)**: Genera automáticamente el valor al insertar
- **IDENTITY**: Usa la estrategia del motor de BD (AUTO_INCREMENT en MySQL, por ejemplo)

## 🔁 Método setEpisodios(...)

```java
public void setEpisodios(List<Episodio> episodios) {
    episodios.forEach(e -> e.setSerie(this));
    this.episodios = episodios;
}
```

Este método:

1. Asegura que cada `Episodio` tenga asignado su `Serie`
2. Luego asigna la lista a la propiedad `this.episodios`

✅ **Esto es esencial para que JPA pueda guardar correctamente los episodios con su respectiva `serie_id`.**

## 🧪 ¿Por qué no aparece el campo serie en la tabla episodios?

Aunque declaras `private Serie serie;`, la columna que JPA genera en la tabla es:

**`serie_id`** (clave foránea).

⚠️ **Si no ves una columna `serie` en tu base de datos, es porque el nombre real es `serie_id`, como resultado del `@ManyToOne`.**

## ✅ Resumen

| Anotación | ¿Qué hace? |
|-----------|------------|
| `@OneToMany` | Relación uno a muchos (una serie → muchos episodios) |
| `mappedBy` | Indica que la otra entidad tiene la clave foránea |
| `cascade = ALL` | Propaga operaciones (guardar, borrar, etc.) |
| `fetch = EAGER` | Carga automática de datos relacionados |
| `@ManyToOne` | Muchos episodios apuntan a una serie |
| `@GeneratedValue` | Autogenera el valor del ID |
| `setEpisodios()` | Asocia los episodios a su serie para guardar bien la relación |