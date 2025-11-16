# ViewModel en Compose — Guía rápida

## 1. Dependencias

En `app/build.gradle.kts` añadir:

```kotlin
dependencies {
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.5.1")
}
```

## 2. Crear `MyUiState` en `ui`

```kotlin
data class MyUiState(

)
```

## 3. Mover variables de `MainActivity` a `MyUiState`

```kotlin
data class MyUiState(
    val revenue: Int = 0,
    val itemsSold: Int = 0,
    val currentItemIndex: Int = 0
)
```
Toda variable que represente estado de UI debe pasar aquí.  
Si es compleja, la lógica se hará en el ViewModel.

## 4. Crear clase `MyViewModel` en `ui`

```kotlin
class MyViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(MyUiState())
    val uiState: StateFlow<MyUiState> = _uiState.asStateFlow()
}
```

## 5. Mover funciones y lógica de negocio de `MainActivity` a `MyViewModel`

```kotlin
class MyViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(MyUiState())
    val uiState: StateFlow<MyUiState> = _uiState.asStateFlow()

    // Por ejemplo:
    fun getCurrentItemPrice(): Int {
        val currentItemIndex = _uiState.value.currentItemIndex
        return Datasource.itemList[currentItemIndex].price
    }
}
```

Algunas funciones podrían necesitar actualizar el estado:

```kotlin
fun onItemClicked() {
    val newRevenue = _uiState.value.revenue + getCurrentItemPrice()
    val itemsSold = _uiState.value.itemsSold + 1
    val items = Datasource.itemList

    // Actualizamos estado con nuevos valores
    _uiState.update { currentState ->
        currentState.copy(
            revenue = newRevenue,
            itemsSold = itemsSold,
            currentItemIndex = items.indexOf(determineItemToShow())
        )
    }
}
```

## 6. Pasar el `MyViewModel` a la pantalla principal (`MyScreen`)
```kotlin
@Composable
fun AppScreen(
    myViewModel: MyViewModel = viewModel()
)
```

## 7. Obtener  `MyUiState` en `MyScreen`
```kotlin
@Composable
fun MyScreen(
    myViewModel: MyViewModel = viewModel()
){
    val myUiState by myViewModel.uiState.collectAsState()
}
```
Así podremos acceder a los estados de las variables.

## 8. Reemplazar antiguas llamadas a las varibles `MyScreen`
Usar `myUiState` para acceder a los valores y `myViewModel` para llamar funciones.
