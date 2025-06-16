# Лабораторна робота №6

## Тема: Реалізація об’єктно-реляційного відображення


## SQL: Створення бази даних (SQL)

```CREATE DATABASE IF NOT EXISTS opendata_db;
USE opendata_db;

CREATE TABLE Dataset (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    author VARCHAR(100),
    created_at DATE
);

CREATE TABLE Category (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE DatasetCategory (
    dataset_id INT,
    category_id INT,
    PRIMARY KEY (dataset_id, category_id),
    FOREIGN KEY (dataset_id) REFERENCES Dataset(id),
    FOREIGN KEY (category_id) REFERENCES Category(id)
);
```

## Bean-класи (Java)
'''
public class Dataset {
    private int id;
    private String title;
    private String description;
    private String author;
    private LocalDate createdAt;
    // Геттери і сеттери
}
'''
### Category.java

```public class Category {
    private int id;
    private String name;
    // Геттери і сеттери
}
```

### DAO-інтерфейси та реалізація

DatasetDAO.java

```public interface DatasetDAO {
    void insert(Dataset dataset);
    Dataset findById(int id);
    List<Dataset> findAll();
}
```
### DatasetDAOImpl.java
```
public class DatasetDAOImpl implements DatasetDAO {
    private Connection conn;

    public DatasetDAOImpl(Connection conn) {
        this.conn = conn;
    }

    @Override
    public void insert(Dataset dataset) {
        String sql = "INSERT INTO Dataset (title, description, author, created_at) VALUES (?, ?, ?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, dataset.getTitle());
            stmt.setString(2, dataset.getDescription());
            stmt.setString(3, dataset.getAuthor());
            stmt.setDate(4, Date.valueOf(dataset.getCreatedAt()));
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Dataset findById(int id) {
        String sql = "SELECT * FROM Dataset WHERE id = ?";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, id);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                Dataset dataset = new Dataset();
                dataset.setId(rs.getInt("id"));
                dataset.setTitle(rs.getString("title"));
                dataset.setDescription(rs.getString("description"));
                dataset.setAuthor(rs.getString("author"));
                dataset.setCreatedAt(rs.getDate("created_at").toLocalDate());
                return dataset;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public List<Dataset> findAll() {
        List<Dataset> list = new ArrayList<>();
        String sql = "SELECT * FROM Dataset";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                Dataset dataset = new Dataset();
                dataset.setId(rs.getInt("id"));
                dataset.setTitle(rs.getString("title"));
                dataset.setDescription(rs.getString("description"));
                dataset.setAuthor(rs.getString("author"));
                dataset.setCreatedAt(rs.getDate("created_at").toLocalDate());
                list.add(dataset);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return list;
    }
}
```

### 4. Тестова програма
Main.java
```
public class Main {
    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/opendata_db", "root", "password")) {
            DatasetDAO datasetDAO = new DatasetDAOImpl(conn);

            Dataset ds = new Dataset();
            ds.setTitle("Energy Statistics");
            ds.setDescription("Energy consumption by region.");
            ds.setAuthor("Ministry of Energy");
            ds.setCreatedAt(LocalDate.now());

            datasetDAO.insert(ds);

            Dataset loaded = datasetDAO.findById(1);
            System.out.println("Завантажено: " + loaded.getTitle());

            System.out.println("Всі набори:");
            for (Dataset d : datasetDAO.findAll()) {
                System.out.println(d.getTitle());
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

![shema](/photo_5368332816896621419_x.jpg)
![shema](/photo_5368605860852528519_y.jpg)

Висновок

Під час виконання роботи було реалізовано повноцінну DAO-інфраструктуру для взаємодії з MySQL базою даних. Створена модель дозволяє ефективно зберігати, зчитувати й обробляти інформацію про відкриті набори даних. DAO-підхід забезпечив зручну архітектуру, що легко масштабується та підтримується.
