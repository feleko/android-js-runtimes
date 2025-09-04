# android-js-runtimes

Packaged Android JavaScriptCore (JSC) runtime (and historically QuickJS). Current maintained artifact: `jsc`.

## Usage

Add JitPack repository in root `settings.gradle`:

```gradle
dependencyResolutionManagement {
	repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
	repositories {
		mavenCentral()
		maven { url 'https://jitpack.io' }
	}
}
```

Dependency:

```gradle
implementation "com.github.feleko:jsc:1.1.0"
```

## Integration – zasady bezpiecznego użycia JSC

Minimalne reguły aby uniknąć crashy typu SIGSEGV w `JSValueCreateJSONString`:

1. Single thread: wszystkie wywołania do JSC przez jeden wątek (HandlerThread / single executor).
2. Lifecycle guard: trzymaj `JSGlobalContextRef` w obiekcie ze stanem `disposed`; po `dispose` blokuj kolejne wywołania.
3. JSValueProtect: każdą wartość przechowywaną poza natychmiastowym zakresem chronić `JSValueProtect` / zwalniać `JSValueUnprotect`.
4. Walidacja przed JSON: sprawdzić typ (`JSValueGetType`) zanim użyjesz `JSValueCreateJSONString`.
5. Brak mieszania wielu wersji: dokładnie jedna biblioteka `libjsc.so` w APK (to repo weryfikuje to taskiem).
6. Fallback: przy błędzie / null z JSON – fallback do serializacji po stronie Dart/Java jeśli dane są prostą strukturą.
7. Sanity test po inicjalizacji: wykonaj prosty `({a:1})` + stringify aby wcześnie wykryć problem z wątkiem/kontekstem.
8. Logging: loguj `threadId`, adres kontekstu i wartość przy kluczowych operacjach (evaluate, stringify).

## Build Guard

Task `verifySingleJscEmbed` failuje build jeśli wykryje więcej niż jedno `android-jsc` w zależnościach (chroni przed ABI conflict).  
W `release` ustawiono `debugSymbolLevel FULL` – ułatwia symbolizację natywnych crashy.

Jeśli build agent (np. JitPack) nie ma zainstalowanego NDK, bundlowanie `libc++_shared.so` jest pomijane (log WARN). Wtedy biblioteka C++ może pochodzić z innej zależności Twojej aplikacji. Jeśli w runtime pojawi się błąd "library libc++\_shared.so not found" – zbuduj projekt z zainstalowanym NDK lokalnie i opublikuj AAR.

### Opcjonalne wyłączenie bundlowania libc++

Jeśli Twoja aplikacja (lub inne AAR-y) już dostarczają `libc++_shared.so`, możesz wyłączyć kopiowanie żeby zmniejszyć ryzyko duplikatów i przyspieszyć build:

```
./gradlew -Pjsc.bundleLibCpp=false :jsc:assembleRelease
```

Domyślnie `jsc.bundleLibCpp=true`.

### Automatyczna weryfikacja obecności libc++
Build task `verifyLibCppBundled` (wykonywany przed publikacją) sprawdza czy w AAR są `jni/<abi>/libc++_shared.so` dla wymaganych ABI (domyślnie `arm64-v8a,armeabi-v7a`). Konfiguracja własnej listy:
```
./gradlew -Pjsc.requiredAbis=arm64-v8a,armeabi-v7a,x86_64 :jsc:verifyLibCppBundled
```

Jeżeli brakuje którejś – build fail.

### Troubleshooting (dlopen failed: libc++_shared.so not found)
1. Upewnij się, że używasz dependency `com.github.feleko:jsc:<version>` z `implementation`, nie `compileOnly`.
2. Rozpakuj AAR (`unzip -l jsc-<version>.aar | grep libc++_shared`) – musi zawierać Twoje ABI.
3. Rozpakuj APK (`unzip -l app-release.apk | grep libc++_shared`) – jeśli brak w APK a był w AAR, sprawdź `packagingOptions` i `jniLibs.srcDirs`.
4. Jeśli świadomie dostarczasz libc++ w innym module, ustaw `-Pjsc.bundleLibCpp=false`.
5. Wymuś fail przy braku: `-Pjsc.failIfNoLibCpp=true`.


### Wymuszenie fail gdy libc++ nie zostanie dołączone

Jeśli chcesz upewnić się, że build NIE przejdzie gdy nie uda się znaleźć NDK / skopiować `libc++_shared.so`:

```
./gradlew -Pjsc.failIfNoLibCpp=true :jsc:assembleRelease
```

## License

MIT
