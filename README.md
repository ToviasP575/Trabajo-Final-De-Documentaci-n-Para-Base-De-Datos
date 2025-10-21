# 🎵 YouTube Music Hits: Top 100 of 2025

## 📖 Descripción general
Este proyecto presenta una base de datos llamada **YouTube Music Hits: Top 100 of 2025**, que recopila las **100 canciones más populares de YouTube en 2025**.  
Cada registro representa un video musical con sus **metadatos esenciales**, incluyendo título, artista/canal, cantidad de visualizaciones, duración, etiquetas (tags), categorías y suscriptores del canal.  

El objetivo de la base de datos es **organizar, analizar y consultar información relevante sobre las tendencias musicales globales en YouTube**, aplicando los principios de normalización y diseño relacional.

La base de datos ha utilizar se extrajo del siguiente enlace: 

```
https://www.kaggle.com/datasets/emanfatima2025/youtube-music-hits-top-100-of-2025?resource=download

```
---

## 🧩 Estructura de la base de datos

### Tablas principales
| Tabla | Descripción | Columnas destacadas |
|-------|--------------|---------------------|
| **channels** | Información de los canales de YouTube | `id`, `name`, `url`, `follower_count` |
| **thumbnails** | URLs de las miniaturas asociadas a los videos | `id`, `url` |
| **songs** | Información principal de las canciones/videos | `id`, `title`, `fulltitle`, `description`, `view_count`, `duration`, `duration_string`, `live_status`, `channel_id`, `thumbnail_id` |
| **categories** | Categorías generales de los videos | `id`, `name` |
| **tags** | Etiquetas o palabras clave asociadas a cada video | `id`, `name` |
| **song_categories** | Relación muchos-a-muchos entre canciones y categorías | `song_id`, `category_id` |
| **song_tags** | Relación muchos-a-muchos entre canciones y etiquetas | `song_id`, `tag_id` |

### Relaciones
- Un **canal** puede tener muchas **canciones**.  
- Cada **canción** tiene un solo **canal** y una sola **miniatura**.  
- Una **canción** puede pertenecer a múltiples **categorías**.  
- Una **canción** puede tener múltiples **tags**.

---

## 🧠 Diagrama Entidad–Relación (E-R)

El siguiente diagrama representa las entidades, atributos y relaciones de la base de datos:  

📎 **Archivo:**

<img width="654" height="705" alt="YoutubeMusic_Diagrama E-R" src="https://github.com/user-attachments/assets/6556646d-c88a-4d2a-9582-4756160225d2" />


---

## 🧮 Normalización

Durante el proceso de diseño se aplicaron las **formas normales hasta la Tercera Forma Normal (3FN)**, garantizando:
- Eliminación de redundancias.  
- Evitación de dependencias parciales o transitivas.  
- Integridad referencial mediante claves foráneas.  

A continuación se detalla el **código SQL utilizado** para la creación, normalización e inserción de datos a partir del archivo CSV original.

---

## 💻 Código SQL completo

```sql
CREATE DATABASE IF NOT EXISTS youtube_top100_2025 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

CREATE TABLE channels (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500),
    follower_count BIGINT,
    UNIQUE KEY (name)
);

CREATE TABLE thumbnails (
    id INT AUTO_INCREMENT PRIMARY KEY,
    url VARCHAR(500) NOT NULL
);

CREATE TABLE songs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    fulltitle VARCHAR(500),
    description TEXT,
    view_count BIGINT,
    duration INT,
    duration_string VARCHAR(50),
    live_status VARCHAR(50),
    channel_id INT,
    thumbnail_id INT,
    FOREIGN KEY (channel_id) REFERENCES channels(id),
    FOREIGN KEY (thumbnail_id) REFERENCES thumbnails(id)
);

CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE song_categories (
    song_id INT,
    category_id INT,
    PRIMARY KEY (song_id, category_id),
    FOREIGN KEY (song_id) REFERENCES songs(id),
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE tags (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE song_tags (
    song_id INT,
    tag_id INT,
    PRIMARY KEY (song_id, tag_id),
    FOREIGN KEY (song_id) REFERENCES songs(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);

CREATE TABLE temp_youtube_data (
    title VARCHAR(500),
    fulltitle VARCHAR(500),
    description TEXT,
    view_count BIGINT,
    categories VARCHAR(255),
    tags TEXT,              
    duration INT,
    duration_string VARCHAR(10),
    live_status VARCHAR(5),
    thumbnail VARCHAR(500),
    channel VARCHAR(100),
    channel_url VARCHAR(500),
    channel_follower_count INT
);

INSERT INTO channels (name, url, follower_count)
SELECT DISTINCT channel, channel_url, channel_follower_count
FROM temp_youtube_data
ON DUPLICATE KEY UPDATE name=name;

INSERT INTO thumbnails (url)
SELECT DISTINCT thumbnail
FROM temp_youtube_data
ON DUPLICATE KEY UPDATE url=url;

INSERT INTO categories (name)
SELECT DISTINCT categories
FROM temp_youtube_data
ON DUPLICATE KEY UPDATE name=name;

INSERT INTO songs (channel_id, thumbnail_id, title, fulltitle, description, view_count, duration, duration_string, live_status)
SELECT
    c.id,
    t.id,
    td.title,
    td.fulltitle,
    td.description,
    td.view_count,
    td.duration,
    td.duration_string,
    td.live_status
FROM temp_youtube_data td
JOIN channels c ON td.channel = c.name
JOIN thumbnails t ON td.thumbnail = t.url;

CREATE TABLE numbers (n INT PRIMARY KEY);
INSERT INTO numbers VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10),
(11), (12), (13), (14), (15), (16), (17), (18), (19), (20),
(21), (22), (23), (24), (25), (26), (27), (28), (29), (30),
(31), (32), (33), (34), (35), (36), (37), (38), (39), (40),
(41), (42), (43), (44), (45), (46), (47), (48), (49), (50);

INSERT IGNORE INTO tags (name)
SELECT
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(td.tags, ';', n.n), ';', -1)) AS tag_name
FROM 
    temp_youtube_data td
JOIN 
    numbers n
    ON CHAR_LENGTH(td.tags) - CHAR_LENGTH(REPLACE(td.tags, ';', '')) >= n.n - 1
WHERE
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(td.tags, ';', n.n), ';', -1)) != '';

INSERT INTO song_tags (song_id, tag_id)
SELECT DISTINCT
    s.id AS song_id,
    t.id AS tag_id
FROM
    temp_youtube_data td
JOIN 
    songs s ON td.title = s.title AND td.channel = (SELECT name FROM channels c WHERE c.id = s.channel_id)
JOIN 
    numbers n ON CHAR_LENGTH(td.tags) - CHAR_LENGTH(REPLACE(td.tags, ';', '')) >= n.n - 1
JOIN 
    tags t ON t.name = TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(td.tags, ';', n.n), ';', -1));

INSERT INTO song_category (song_id, category_id)
SELECT DISTINCT
    s.id AS song_id,
    ct.id AS category_id
FROM
    temp_youtube_data td
JOIN 
    songs s ON td.title = s.title AND td.channel = (SELECT name FROM channels c WHERE c.id = s.channel_id)
JOIN 
    categories ct ON TRIM(td.categories) = ct.name;
