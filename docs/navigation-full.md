# Navegación en Compose — Guía rápida

## 1. Dependencias
En `app/build.gradle.kts` añadir:
```kotlin
dependencies {
    implementation("androidx.navigation:navigation-compose:2.7.4")
}
```
## 2. Crear `enum class MyScreen` en la pantalla principal (`MyScreen`)
Creamos una enum class que actúa como fuente única de verdad para todas las rutas de la app.
Incluye el título de cada pantalla mediante recursos `@StringRes`.
Se coloca al inicio del composable raíz, tras los imports.
```kotlin
enum class MyScreen(@StringRes val title: Int) {
    Start(title = R.string.start_screen),
    Second(title = R.string.second_screen),
    Third(title = R.string.third_screen)
}
```

## 3. En el composable principal pasamos `MyViewModel` y `NavHostController`
```kotlin
@Composable
fun MyScreen(
    viewModel: MyViewModel = viewModel(),
    navController: NavHostController = rememberNavController()
) {
    // Pila de actividades y pantalla actual
    val backStackEntry by navController.currentBackStackEntryAsState()
    val currentScreen = MyScreen.valueOf(backStackEntry?.destination?.route ?: AppScreen.Start.name)
}
```
También inicializamos la entrada de la pila de actividades y el nombre de la pantalla actual.

## 4. Creamos el composable `MyAppBar` y lo preparamos para navegación
Se define una TopAppBar que muestra el título de la pantalla actual.
Incluye un botón de retroceso cuando es posible navegar atrás.
Recibe: `currentScreen`, `canNavigateBack`, `navigateUp`.
```kotlin
@Composable
fun MyAppBar(
    currentScreen: MyScreen,
    canNavigateBack: Boolean,
    navigateUp: () -> Unit,
    modifier: Modifier = Modifier
) {
    TopAppBar(
        title = { Text(stringResource(currentScreen.title)) },
        colors = TopAppBarDefaults.mediumTopAppBarColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer
        ),
        modifier = modifier,
        navigationIcon = {
            if (canNavigateBack) {
                IconButton(onClick = navigateUp) {
                    Icon(
                        imageVector = Icons.Filled.ArrowBack,
                        contentDescription = stringResource(R.string.back_button)
                    )
                }
            }
        }
    )
}
```

## 5. Añadir `MyAppBar` a la `topBar` del `Scaffold` raíz
Incorporamos `MyAppBar` al parámetro `topBar` del `Scaffold` del composable raíz.
Esto garantiza que la barra superior se mantenga visible en todas las pantallas y reaccione automáticamente a los cambios de navegación.
```kotlin
@Composable
fun MyScreen(
    viewModel: MyViewModel = viewModel(),
    navController: NavHostController = rememberNavController()
) {

    val backStackEntry by navController.currentBackStackEntryAsState()
    val currentScreen = MyScreen.valueOf(backStackEntry?.destination?.route ?: AppScreen.Start.name)

    Scaffold(
        topBar = {
            MyAppBar(
                currentScreen = currentScreen,
                canNavigateBack = navController.previousBackStackEntry != null, // Importante
                navigateUp = { navController.navigateUp() })
        }
    ) {
        innerPadding ->

        // NavHost
    }
}
```

## 6. Definir el `NavHost` dentro del `Scaffold` e instanciar `MyUiState`
```kotlin
Scaffold(
        topBar = {
            MyAppBar(
                currentScreen = currentScreen,
                canNavigateBack = navController.previousBackStackEntry != null,
                navigateUp = { navController.navigateUp() })
        }
    ) {
        innerPadding ->
        val uiState by viewModel.uiState.collectAsState()

        // NavHost
        NavHost(
            navController = navController,
            startDestination = LunchTrayScreen.Start.name,
            modifier = Modifier.padding(innerPadding)
        ){

        }
    }
```

## 7. Declarar las rutas de la app
Dentro del `NavHost` definimos todas las pantallas usando `composable(route)`.
Cada entrada asocia la ruta de `MyScreen` con su composable correspondiente.
```kotlin
NavHost(
            navController = navController,
            startDestination = LunchTrayScreen.Start.name,
            modifier = Modifier.padding(innerPadding)
        ){
            composable(rout = MyScreen.Start.name){
                // Componible de la StartScreen
            }

            composable(rout = MyScreen.Second.name){
                // Componible de la SecondScreen
            }

            composable(rout = MyScreen.Third.name){
                // Componible de la ThirdScreen
            }
        }
```

## 8. Añadir funciones de navegación a los botones
Para avanzar a la siguiente pantalla:
```kotlin
onNextButtonClicked = { navController.navigate(MyScreen.NextScreen.name) }
```
Ejemplo de función que resetea el estado y vuelve a la primera pantalla:
```kotlin
private fun cancelAndNavigateToStart(
    viewModel: MyViewModel,
    navController: NavHostController
) {
    viewModel.resetState()
    navController.popBackStack(MyScreen.Start.name, false) // Importante
}
```
Estas funciones pueden combinarse con lógica del `ViewModel` para actualizar estado antes de navegar.