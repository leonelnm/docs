# Básico

## Abrir nueva Activity
```java
Intent intent = new Intent(this, SecondActivity.class); // La clase que gestiona el activity
startActivity(intent);
```

## Crear un Shared Preferences
Se pueden crear todos los archivos de preferencias compartidas.
+ Crear archivo, si no existe se crea un archivo vacío
```java
SharedPreferences preferences = getSharedPreferences(filename, Context.MODE_PRIVATE);
```
+ Guardar clave-valor en el archivo de preferencias
  + Métodos para guardar datos, parar recuperar utilizar get`TIPO`:
    + `putInt("keyname", 5)`
    + `putString("keyname", "data")`
    + `putBoolean("keyname", true)`
    + `putFloat("keyname", 1f)`
    + `putStringSet("keyname", Set values)`
```java
SharedPreferences preferences = getSharedPreferences(filename, Context.MODE_PRIVATE);
SharedPreferences.Editor editorPreferences = preferences.edit();
editorPreferences.putString("keyname", "valor");

editorPreferences.apply(); //Commit (persiste los datos)
```
+ Recuperar valor de preferencias
```java
SharedPreferences preferences = getSharedPreferences(filename, Context.MODE_PRIVATE);
preferences.getString("keyname", "default");
```