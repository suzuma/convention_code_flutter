# Coding Convention (Dart - Flutter)

- Classes, enum types, typedefs, y se utilizan parámetros de tipo `UpperCamelCase`.

```dart
class SliderMenu { ... }

class HttpRequest { ... }

typedef Predicate<T> = bool Function(T value);
```

- Nombres de bibliotecas, paquetes, directorios y archivos fuente utilizados. `lowercase_with_underscores`.

```dart
library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';
```

-Usar en otros casos `lowerCamelCase`.

```dart
var count = 3;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```

- Usar `single quote`

```dart
var foo = 'bar'; // Good
var foo = "bar"; // Bad
```
- No utilice la lista para nombrar variables

```dart
List<UserRes> users =[]; // Good
List<UserRes> listUser =[]; // Bad
```
- Absolute imports (importacion de paquetes)

```dart
import 'package:new_vime/feature/authenticate/domain/repository/authenticate_repository.dart';// Good
import '../../repository/authenticate_repository.dart'; // Bad
```
Declarar variables en la pantalla por Bloc, Variable
```dart
 //Bloc
  late UserBloc userBloc;
  late EditUserBloc editUserBloc;
  //Variable
  String? id;
  String? domain;
```
- El comentario describe las funciones de  `Functions`, `Properties`.

````dart
/// URL avatar.
///
/// ```dart
/// var example = CodeBlockExample();
/// print(example.isItGreat); // "Yes."
/// ```
/// ABC XYZ.
String? avatarPath;
````
### Bloc

##### Bloc Events

- Empezar con **Verd**
- Termina con sufijo **Event**

Example:
```
- ToDosEvent
- LoadToDosEvent
- DeleteToDoEvent
```

##### Bloc States

Example:
```
- Loading
- Loaded
- Failed
```
### Arquitectura limpia

![image](https://github.com/user-attachments/assets/f191eeac-0348-45e4-8ef6-3673612afcf3)

Utilice 3 capas: Presentation, Domain and Data.

### Capa de Presentation(UI)
Ở tầng này chúng ta sẽ tổ chức code theo feature-first, sẽ bao gồm bloc, widget, screen. Chức năng chính của **Presentation** là: 

- Quản lý trạng thái(State management)
- Xử lý UI của ứng dụng, hiển thị dữ liệu,...

````dart
class LoginBloc extends Bloc<LoginEvent, LoginState> {
  LoginBloc() : super(LoginInitial()) {
    on<LoginWithUserNamePassWordEvent>((event, emit) async {
      emit(LoginLoading());
      try {
        var useCase = await GetIt.I.getAsync<LoginUseCase>();
        final function = await useCase.call(ParamsLoginUseCase(userName: event.userName, passWord: event.passWord));
        SmartDialog.showLoading(msg: 'loading'.tr());
        function.fold((failure) {
          SmartDialog.dismiss();
          if (failure is ConnectionFailure) {
            SmartDialog.showNotify(msg: 'noInternetConnection'.tr(), notifyType: NotifyType.warning);
          } else {
            emit(const LoginFailed(error: Constants.login));
          }
        }, (data) async {
          emit(const LoginLoaded());
        });
      } catch (e) {
        SmartDialog.dismiss();
        emit(LoginFailed(error: e.toString()));
      }
    });
  }
}
````
### Capa de Dominio

En esta capa de Dominio, tendremos lo siguiente:

    Implementación de casos de uso
    Implementación de interfaces de repositorios

Caso de uso: Dependiendo de la funcionalidad, se puede implementar o no un caso de uso.

Ejemplo: Para ilustrar la utilidad de los casos de uso, consideremos la lógica de cierre de sesión de una cuenta. Aquí, se deben realizar tareas como eliminar la información almacenada del usuario y cancelar la suscripción a FCM (Firebase Cloud Messaging). En la aplicación, hay dos lugares donde se puede hacer clic para cerrar sesión: uno en la página de perfil y otro en la página de inicio, que también tiene un botón para cerrar sesión. En situaciones como esta, es simple reutilizar una única función de manejo de casos de uso de cierre de sesión (LogoutUseCases). Un caso de uso siempre define claramente las entradas y salidas, lo que facilita su prueba en esta etapa.

![image](https://github.com/user-attachments/assets/bfda5115-7f4e-4c52-980b-b16b193023f6)

**Repository**: Tầng domain chỉ chứa các interface cho repo còn việc thực thi chúng thì nằm ở tầng data
````dart
abstract class AuthenticateRepository {
  Future<Either<Failure, void>> login(String username, String passWord);
  Future<Either<Failure, void>> signUp(String username, String passWord, String email);
};
````
### Tầng Data
Tầng Data thường được chia thành ba phần chính:

**Repositories**: Là các lớp triển khai cụ thể của các abstract repositories, implements phương thức định nghĩa trong abstract repositories.
````dart
class LoginRepositoryImpl implements AuthenticateRepository {
  final AuthenticateDataSource authenticateDataSource;
  final NetworkInfo networkInfo;
  LoginRepositoryImpl({required this.authenticateDataSource, required this.networkInfo});
  @override
  Future<Either<Failure, void>> login(String username, String passWord) async {
    final isConnected = await networkInfo.isConnected;
    if (isConnected) {
      try {
        final result = await authenticateDataSource.login(username, passWord);
        return Right(result);
      } on ServerException {
        return Left(ServerFailure());
      }
    } else {
      return Left(ConnectionFailure());
    }
  }
}
````
**Data Sources**:

Remote Data Source: Chịu trách nhiệm làm việc với các nguồn dữ liệu từ xa, như API RESTful, ...

Local Data Source: Chịu trách nhiệm làm việc với các nguồn dữ liệu cục bộ như SQLite, SharedPreferences,...
````dart
abstract class AuthenticateDataSource {
  Future<void> login(String userName, String passWord);
  Future<void> signUp(String phoneNumber, String passWord, String email);
}

class AuthenticateDataSourceImpl implements AuthenticateDataSource {
  late final Dio dio;
  AuthenticateDataSourceImpl({required this.dio});
  @override
  Future<void> login(String userName, String passWord) async {
    final response = await dio.post(Constants.login,
        data: {'userName': userName, 'password': passWord});

    await setValueString(SharedPrefKey.domain, Constants.domain);
    await setValueString(
        SharedPrefKey.tokenUser, response.data['data']['accessToken']);
    await setValueString(
        SharedPrefKey.idUser, response.data['data']['user']['id']);
    print(response.data['data']['accessToken']);
  }
````

**Models (Data Models)**: Map json sang model sử dụng ![json_serializable](https://pub.dev/packages/json_serializable) để generate code.

Dựa vào type trả về của API để đặt tên cho model. Ví dụ: UserRes, UserReq,...
## Cấu trúc source

>

      lib
  ├── app
  ├── common
  │   ├── config
  │   ├── constant                                  # Variables compartidas
  │   ├── model                                     # Modelos de toda la aplicación
  │   ├── preference                                # Funciones de shared_preference
  │   ├── route                                     # Rutas para toda la aplicación
  │   ├── util                                      # Funciones reutilizables
  │   └── widget                                    # Definición de widgets compartidos
  ├── feature                                       # Lista de funcionalidades de la aplicación
  │   ├── login
  │   │   ├── data
  │   │   │   ├── data_sources
  │   │   │   ├── repository                        # Implementación del repositorio
  │   │   │   ├── model
  │   │   ├── domain
  │   │   │   ├── usecases
  │   │   │   ├── repository                        # Clase abstracta del repositorio
  │   │   ├── presentation
  │   │   │   ├── bloc                              # Bloc de la funcionalidad (varios blocs diferentes)
  │   │   │   │   ├── export.dart                   # Exportación compartida del Bloc
  │   │   │   │   ├── login_bloc.dart               # Bloc que maneja la funcionalidad de inicio de sesión
  │   │   │   │   ├── login_event.dart              # Eventos del Bloc para la funcionalidad de inicio de sesión
  │   │   │   │   ├── login_state.dart              # Estados del Bloc para la funcionalidad de inicio de sesión
  │   │   │   ├── screen
  │   │   │   │   ├── login_screen.dart 
  │   │   │   │   ├── widget
  │   │   │   │   │   ├── button_login_widget.dart  # Widget de la pantalla de inicio de sesión
  │   └── ...
  └── translations                                  # Configuración multilingüe

