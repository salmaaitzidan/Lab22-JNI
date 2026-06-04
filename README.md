# JNIDemo — Application Android avec Java Native Interface (JNI)

Projet Android développé dans le cadre d'un laboratoire pédagogique sur l'utilisation du NDK Android et de JNI pour faire communiquer du code Java avec du code natif C++.

---

## Objectif du projet

L'application **JNIDemo** montre comment une application Android peut appeler des fonctions écrites en C++ directement depuis Java, via le mécanisme **JNI (Java Native Interface)**. Elle illustre :

- le passage de types simples (`int`, `String`) entre Java et C++ ;
- le passage d'un tableau `int[]` vers le natif ;
- la gestion des erreurs côté C++ (valeurs négatives, overflow) ;
- l'utilisation des logs natifs avec `__android_log_print` (visible dans Logcat) ;
- la bonne gestion des ressources JNI (`ReleaseStringUTFChars`, `ReleaseIntArrayElements`).

---

## Fonctionnalités

| Fonctionnalité | Description |
|---|---|
| `helloFromJNI()` | Retourne une chaîne de caractères construite en C++ |
| `factorial(int n)` | Calcule n! en C++ avec gestion d'overflow et de valeur négative |
| `reverseString(String s)` | Inverse une chaîne Java côté C++ avec `std::reverse` |
| `sumArray(int[] values)` | Calcule la somme d'un tableau d'entiers en C++ |

---

## Architecture du projet

```
JNIDemo/
├── app/
│   └── src/
│       └── main/
│           ├── cpp/
│           │   ├── CMakeLists.txt        ← configuration de compilation native
│           │   └── native-lib.cpp        ← code C++ avec les fonctions JNI
│           ├── java/com/example/jnidemo/
│           │   └── MainActivity.java     ← activité principale, appels natifs
│           ├── res/
│           │   ├── layout/
│           │   │   └── activity_main.xml ← interface utilisateur
│           │   └── values/
│           │       ├── strings.xml
│           │       └── themes.xml
│           └── AndroidManifest.xml
└── build.gradle
```

---

## Flux d'exécution JNI

```
Java (MainActivity)
        │
        │  appel méthode native
        ▼
System.loadLibrary("native-lib")
        │
        │  charge libnative-lib.so
        ▼
Code C++ (native-lib.cpp)
        │
        │  traitement + retour
        ▼
Java (affichage du résultat)
```

---

## Prérequis

- Android Studio Hedgehog ou plus récent
- NDK installé (via SDK Manager → SDK Tools → NDK Side by side)
- CMake installé (via SDK Manager → SDK Tools → CMake)
- SDK Android API 24 minimum
- Java 8

---

## Installation et exécution

1. **Cloner le dépôt :**
   ```bash
   git clone https://github.com/<votre-username>/JNIDemo.git
   cd JNIDemo
   ```

2. **Ouvrir dans Android Studio :**
   File → Open → sélectionner le dossier `JNIDemo`

3. **Attendre la synchronisation Gradle** (peut prendre quelques minutes à la première ouverture)

4. **Vérifier que NDK et CMake sont installés :**
   Tools → SDK Manager → SDK Tools → cocher NDK et CMake

5. **Lancer l'application :**
   Run → Run 'app' sur un émulateur (API 24+) ou un appareil réel

---

## Explication du code natif (`native-lib.cpp`)

### Convention de nommage JNI

Le nom de chaque fonction C++ suit la convention :

```
Java_<package>_<Classe>_<méthode>
```

Exemple :
```cpp
Java_com_example_jnidemo_MainActivity_helloFromJNI
```

Si le package ou le nom de classe change, la signature native doit être mise à jour, sinon Android lancera une `UnsatisfiedLinkError` au moment de l'appel.

### Chargement de la bibliothèque

```java
static {
    System.loadLibrary("native-lib");
}
```

Ce bloc statique charge la bibliothèque `libnative-lib.so` au moment du chargement de la classe. Le préfixe `lib` et le suffixe `.so` sont ajoutés automatiquement par Android.

### Gestion des chaînes

```cpp
const char* chars = env->GetStringUTFChars(javaString, nullptr);
// ... utilisation ...
env->ReleaseStringUTFChars(javaString, chars);
```

Il est obligatoire de libérer les ressources JNI après usage pour éviter les fuites mémoire.

### Logs natifs

```cpp
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, "JNI_DEMO", __VA_ARGS__)
```

Les logs natifs sont visibles dans **Logcat** sous le tag `JNI_DEMO`.

---

## Gestion des erreurs

| Cas | Valeur retournée |
|---|---|
| Factoriel d'un nombre négatif | `-1` |
| Overflow (résultat > INT_MAX) | `-2` |
| Tableau nul | `-1` |
| Impossible d'accéder aux éléments | `-2` |

---

## Tests à effectuer

| Test | Entrée | Résultat attendu |
|---|---|---|
| Factoriel normal | `10` | `3628800` |
| Factoriel négatif | `-5` | Erreur `-1` |
| Factoriel overflow | `20` | Erreur `-2` |
| Inversion de chaîne | `JNI is powerful!` | `!lufrewop si INJ` |
| Chaîne vide | `""` | `""` |
| Somme tableau | `10, 20, 30, 40, 50` | `150` |
| Tableau vide | *(champ vide)* | `0` |

---

## Lire les logs natifs dans Logcat

1. Lancer l'application
2. Ouvrir Logcat : View → Tool Windows → Logcat
3. Filtrer par tag : `JNI_DEMO`

Messages attendus :
```
I/JNI_DEMO: Appel de helloFromJNI depuis le natif
I/JNI_DEMO: Factoriel de 10 calcule en natif = 3628800
I/JNI_DEMO: String inversee = !lufrewop si INJ
I/JNI_DEMO: Somme du tableau = 150
```

---

## Bonnes pratiques JNI appliquées

- Libération systématique des ressources (`Release*`)
- Vérification des pointeurs nuls avant usage
- Gestion des overflows sur les types entiers
- Logs natifs avec tag pour faciliter le diagnostic
- Nom de bibliothèque cohérent entre `CMakeLists.txt` et `loadLibrary()`

---

## Technologies utilisées

- **Java** — logique principale et interface Android
- **C++17** — fonctions natives via JNI
- **CMake 3.22.1** — système de build natif
- **Android NDK** — compilation du code C++ pour Android
- **JNI (Java Native Interface)** — pont entre Java et C++
- **Android Logcat** — journalisation des appels natifs

---
