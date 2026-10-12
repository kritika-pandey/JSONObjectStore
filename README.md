# JSONObjectStore

A lightweight Python library that saves custom objects to JSON files and loads them back as working objects. It offers a flat key-value store and a named-table store, built from scratch with only the Python standard library, inside a single Jupyter notebook.

## Features

- Save custom Python objects to a JSON file and reload them after the program ends
- Two storage styles built on one shared base class:
  - `KVStore`: flat key to value storage
  - `DBTable`: records grouped under a named table
- Input validation (keys and table names must be strings)
- Rebuild a live object from saved data with `Hector(**saved_dictionary)`
- No external dependencies

## Requirements

- Python 3.6 or higher (uses f-strings)
- Jupyter Notebook, JupyterLab, or any notebook environment (Google Colab works too)

## Notebook layout

Run the cells in this order. Cells 3 and 4 depend on the classes defined in Cells 1 and 2.

| Cell | Contents | Purpose |
|---|---|---|
| 1 | Validation helpers, `Database`, `KVStore`, `DBTable` | The storage library |
| 2 | `Car`, `Hector` | Sample objects to store |
| 3 | Key-value store demo | Save, reload, list, count and delete objects |
| 4 | Table store demo | Insert several objects into a named table and save |

## Getting started

1. Open the notebook.
2. Run Cell 1, then Cell 2, then Cell 3 and Cell 4.
3. The demos create two files in the notebook's working folder: `my-kv_store` and `my-cars`.

## Usage

### Key-value store

```python
h1 = Hector()
h1.speed_up(40)

store = KVStore("my-kv_store")      # loads the file if it exists
store.set("Hector", h1)             # stored in memory
store.save()                        # written to disk

data = store.get("Hector")          # {'current_state': 'Running', 'current_speed': 50}
h2 = Hector(**data)                 # a new, working Hector object
h2.show_state()                     # Hector is Running and current speed is 50
```

### Table store

```python
t1 = Hector()
t1.speed_up(40)

cars = DBTable("CARS", "my-cars")
cars.insert("h1", t1)
cars.save()

print(cars.db)
# {'CARS': {'h1': '{"current_state": "Running", "current_speed": 50}'}}
```

## API reference

### `Database` (base class)

| Method | Description |
|---|---|
| `Database(location=None)` | Creates the database. If a path is given, loads the file when it exists, otherwise starts empty. |
| `save(location=None)` | Writes the data to disk as JSON. A path passed here overrides the stored one. Raises `ValueError` if no path is available. |

### `KVStore`

| Method | Description |
|---|---|
| `set(key, value)` | Stores an object's attributes (`value.__dict__`) under `key`. Overwrites an existing key. |
| `get(key)` | Returns the saved attributes as a dictionary. Raises `KeyError` if the key is missing. |
| `delete(key)` | Removes the key. Does nothing if it doesn't exist. |
| `exists(key)` | Returns `True` or `False`. |
| `keys()` | Returns all stored keys. |
| `count()` | Returns the number of stored entries. |
| `key_count()` | Alias for `count()`. |

### `DBTable`

| Method | Description |
|---|---|
| `DBTable(table_name, location=None)` | Creates the database and an empty table with the given name. |
| `insert(key, value)` | Stores an object's attributes under `key` inside the table. |
| `select(key)` | Returns one record by key. |
| `select_all()` | Returns every record in the table. |
| `delete(key)` | Removes a record. |
| `exists(key)` | Returns `True` or `False`. |
| `count()` | Returns the number of records in the table. |

### Vehicles (sample data)

| Class | Description |
|---|---|
| `Car` | Tracks `current_state` (`"Standing"` or `"Running"`) and `current_speed`. Methods: `start()`, `stop()`, `speed_up(n)`, `speed_down(n)`, `show_state()`. |
| `Hector(Car)` | Inherits from `Car` and overrides `show_state()` with a Hector-specific message. |

## How it works

1. JSON can't store a custom object directly, so `set()` and `insert()` save the object's attributes (`value.__dict__`) as a JSON string.
2. `save()` writes the whole in-memory dictionary to the file with `json.dump()`.
3. When a store is created with the same path, `_load_()` reads the file back into memory.
4. `get()` returns the saved attributes as a dictionary, and `Hector(**data)` unpacks it into the constructor to rebuild a working object.

Changes made with `set()`, `insert()` or `delete()` only live in memory until `save()` is called.

## Notebook tips and notes

- **Run cells in order.** Running Cell 3 or 4 before Cells 1 and 2 raises a `NameError`.
- **After editing a class, re-run its cell and the cells after it.** Objects created earlier keep using the old version of the class.
- **After restarting the kernel, run all cells again from Cell 1.** Only the saved files stay on disk.
- **For a clean start, delete `my-kv_store` and `my-cars`.** A store loads any existing file with the same name, so old data can appear in the output.
- **Re-running Cell 3 reloads the saved file.** The demo deletes the `"Hector"` entry in memory but doesn't call `save()` afterwards, so the file on disk still contains it.
- **Only an object's attributes are stored, not its class.** The class must be supplied again when rebuilding the object.

## Concepts practiced

Classes and inheritance, `super()`, file handling, JSON serialization, dictionary unpacking (`**`), and input validation.
