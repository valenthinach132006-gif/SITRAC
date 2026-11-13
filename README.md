
 #include <iostream>
#include <string>
#include <sstream>
#include <fstream>
#include <iomanip>
#include <limits>
#include <vector>

using namespace std;


// 1. ESTRUCTURAS DE DATOS


struct Arista {
    int destinoId;
    double distancia;
    Arista* siguiente;
};

struct Ciudad {
    int id;
    string nombre;
    Arista* adyacencias;
    Ciudad* siguiente;
};

struct Pasajero {
    int id;
    string nombre;
    string telefono;
    Pasajero* siguiente;
};

struct Solicitud {
    int id;
    int pasajeroId;
    int origenId;
    int destinoId;
    Solicitud* siguiente;
};


// 2. VARIABLES GLOBALES DEL SISTEMA


Ciudad* listaCiudades = NULL;       // Los vertices del grafo
Pasajero* listaPasajeros = NULL;    // Lista enlazada de pasajeros
Solicitud* frenteCola = NULL;      // Frente de la cola
Solicitud* finalCola = NULL;       // Final de la cola

int siguienteIdCiudad = 1;
int siguienteIdPasajero = 1;
int siguienteIdSolicitud = 1;

vector<string> historial;


// 3. FUNCIONES GENERALES DE ENTRADA Y APOYO


string leerTexto(const string& mensaje) {
    string texto;

    while (true) {
        cout << mensaje;
        getline(cin, texto);

        if (texto.find_first_not_of(" \t\r\n") != string::npos) {
            return texto;
        }

        cout << "ERROR: el texto no puede estar vacio.\n";
    }
}

int leerEntero(const string& mensaje) {
    string linea;
    int valor;
    char sobrante;

    while (true) {
        cout << mensaje;
        getline(cin, linea);

        stringstream entrada(linea);

        if (entrada >> valor && !(entrada >> sobrante)) {
            return valor;
        }

        cout << "ERROR: ingrese un numero entero valido.\n";
    }
}

double leerReal(const string& mensaje) {
    string linea;
    double valor;
    char sobrante;

    while (true) {
        cout << mensaje;
        getline(cin, linea);

        stringstream entrada(linea);

        if (entrada >> valor && !(entrada >> sobrante)) {
            return valor;
        }

        cout << "ERROR: ingrese un numero valido.\n";
    }
}

void registrarHistorial(const string& evento) {
    historial.push_back(evento);
}

Ciudad* buscarCiudadPorId(int id) {
    Ciudad* actual = listaCiudades;

    while (actual != NULL) {
        if (actual->id == id) {
            return actual;
        }
        actual = actual->siguiente;
    }

    return NULL;
}

Ciudad* buscarCiudadPorNombre(const string& nombre) {
    Ciudad* actual = listaCiudades;

    while (actual != NULL) {
        if (actual->nombre == nombre) {
            return actual;
        }
        actual = actual->siguiente;
    }

    return NULL;
}

Pasajero* buscarPasajeroPorId(int id) {
    Pasajero* actual = listaPasajeros;

    while (actual != NULL) {
        if (actual->id == id) {
            return actual;
        }
        actual = actual->siguiente;
    }

    return NULL;
}

bool existeRuta(int origenId, int destinoId) {
    Ciudad* origen = buscarCiudadPorId(origenId);

    if (origen == NULL) {
        return false;
    }

    Arista* arista = origen->adyacencias;

    while (arista != NULL) {
        if (arista->destinoId == destinoId) {
            return true;
        }
        arista = arista->siguiente;
    }

    return false;
}

void insertarArista(Ciudad* origen, int destinoId, double distancia) {
    Arista* nueva = new Arista;
    nueva->destinoId = destinoId;
    nueva->distancia = distancia;
    nueva->siguiente = origen->adyacencias;
    origen->adyacencias = nueva;
}

void eliminarAristasDeCiudad(Ciudad* ciudad) {
    Arista* actual = ciudad->adyacencias;

    while (actual != NULL) {
        Arista* borrar = actual;
        actual = actual->siguiente;
        delete borrar;
    }

    ciudad->adyacencias = NULL;
}

int contarCiudades() {
    int total = 0;
    Ciudad* actual = listaCiudades;

    while (actual != NULL) {
        total++;
        actual = actual->siguiente;
    }

    return total;
}

int contarPasajeros() {
    int total = 0;
    Pasajero* actual = listaPasajeros;

    while (actual != NULL) {
        total++;
        actual = actual->siguiente;
    }

    return total;
}

int contarSolicitudes() {
    int total = 0;
    Solicitud* actual = frenteCola;

    while (actual != NULL) {
        total++;
        actual = actual->siguiente;
    }

    return total;
}

int contarRutas() {
    int total = 0;
    Ciudad* ciudad = listaCiudades;

    while (ciudad != NULL) {
        Arista* arista = ciudad->adyacencias;

        while (arista != NULL) {
            if (ciudad->id < arista->destinoId) {
                total++;
            }
            arista = arista->siguiente;
        }

        ciudad = ciudad->siguiente;
    }

    return total;
}

void mostrarCiudadesEnConsola() {
    Ciudad* actual = listaCiudades;

    if (actual == NULL) {
        cout << "No hay ciudades registradas.\n";
        return;
    }

    cout << "\n CIUDADES REGISTRADAS \n";

    while (actual != NULL) {
        cout << "ID: " << actual->id
             << " | Nombre: " << actual->nombre << "\n";
        actual = actual->siguiente;
    }
}


// 4. MODULO DE CIUDADES Y GRAFO


void registrarCiudad() {
    cout << "\n===== REGISTRAR CIUDAD =====\n";

    string nombre = leerTexto("Nombre de la ciudad: ");

    if (buscarCiudadPorNombre(nombre) != NULL) {
        cout << "ERROR: esa ciudad ya esta registrada.\n";
        return;
    }

    Ciudad* nueva = new Ciudad;
    nueva->id = siguienteIdCiudad++;
    nueva->nombre = nombre;
    nueva->adyacencias = NULL;
    nueva->siguiente = NULL;

    if (listaCiudades == NULL) {
        listaCiudades = nueva;
    } else {
        Ciudad* ultimo = listaCiudades;

        while (ultimo->siguiente != NULL) {
            ultimo = ultimo->siguiente;
        }

        ultimo->siguiente = nueva;
    }

    cout << "Ciudad registrada correctamente. ID asignado: "
         << nueva->id << "\n";

    registrarHistorial("Se registro la ciudad: " + nombre);
}

void listarCiudades() {
    mostrarCiudadesEnConsola();
}

void buscarCiudad() {
    cout << "\n===== BUSCAR CIUDAD =====\n";

    int id = leerEntero("Ingrese el ID de la ciudad: ");
    Ciudad* ciudad = buscarCiudadPorId(id);

    if (ciudad == NULL) {
        cout << "No existe una ciudad con ese ID.\n";
        return;
    }

    cout << "Ciudad encontrada: ID " << ciudad->id
         << " - " << ciudad->nombre << "\n";

    cout << "Ciudades conectadas:\n";

    Arista* arista = ciudad->adyacencias;

    if (arista == NULL) {
        cout << "  No tiene conexiones registradas.\n";
    }

    while (arista != NULL) {
        Ciudad* destino = buscarCiudadPorId(arista->destinoId);

        if (destino != NULL) {
            cout << "  " << destino->nombre
                 << " (ID " << destino->id << ")"
                 << " - Distancia: " << arista->distancia << "\n";
        }

        arista = arista->siguiente;
    }
}

void quitarConexionEntre(int origenId, int destinoId) {
    Ciudad* origen = buscarCiudadPorId(origenId);

    if (origen == NULL) {
        return;
    }

    Arista* actual = origen->adyacencias;
    Arista* anterior = NULL;

    while (actual != NULL) {
        if (actual->destinoId == destinoId) {
            if (anterior == NULL) {
                origen->adyacencias = actual->siguiente;
            } else {
                anterior->siguiente = actual->siguiente;
            }

            delete actual;
            return;
        }

        anterior = actual;
        actual = actual->siguiente;
    }
}

void eliminarCiudad() {
    cout << "\n ELIMINAR CIUDAD \n";

    int id = leerEntero("ID de la ciudad que desea eliminar: ");

    Ciudad* actual = listaCiudades;
    Ciudad* anterior = NULL;

    while (actual != NULL && actual->id != id) {
        anterior = actual;
        actual = actual->siguiente;
    }

    if (actual == NULL) {
        cout << "ERROR: la ciudad no existe.\n";
        return;
    }

    string nombreEliminado = actual->nombre;

    // Quitar todas las aristas que llegan a esta ciudad.
    Ciudad* otra = listaCiudades;

    while (otra != NULL) {
        if (otra->id != id) {
            quitarConexionEntre(otra->id, id);
        }
        otra = otra->siguiente;
    }

    // Liberar las aristas que salen de la ciudad.
    eliminarAristasDeCiudad(actual);

    // Desconectar el nodo de la lista de ciudades.
    if (anterior == NULL) {
        listaCiudades = actual->siguiente;
    } else {
        anterior->siguiente = actual->siguiente;
    }

    delete actual;

    cout << "Ciudad eliminada. Sus conexiones tambien fueron retiradas.\n";
    registrarHistorial("Se elimino la ciudad: " + nombreEliminado);
}

void crearRuta() {
    cout << "\n CREAR RUTA \n";

    mostrarCiudadesEnConsola();

    int origenId = leerEntero("ID de la ciudad de origen: ");
    int destinoId = leerEntero("ID de la ciudad de destino: ");

    if (origenId == destinoId) {
        cout << "ERROR: una ciudad no puede conectarse consigo misma.\n";
        return;
    }

    Ciudad* origen = buscarCiudadPorId(origenId);
    Ciudad* destino = buscarCiudadPorId(destinoId);

    if (origen == NULL || destino == NULL) {
        cout << "ERROR: una o ambas ciudades no existen.\n";
        return;
    }

    if (existeRuta(origenId, destinoId)) {
        cout << "ERROR: esa ruta ya existe.\n";
        return;
    }

    double distancia = leerReal("Distancia de la ruta (km): ");

    if (distancia < 0) {
        cout << "ERROR: la distancia no puede ser negativa.\n";
        return;
    }

    // Se crea una conexion no dirigida: A-B y B-A.
    insertarArista(origen, destinoId, distancia);
    insertarArista(destino, origenId, distancia);

    cout << "Ruta creada correctamente entre "
         << origen->nombre << " y " << destino->nombre << ".\n";

    registrarHistorial("Se creo una ruta entre " +
                       origen->nombre + " y " + destino->nombre);
}

void eliminarRuta() {
    cout << "\n ELIMINAR RUTA \n";

    int origenId = leerEntero("ID de ciudad de origen: ");
    int destinoId = leerEntero("ID de ciudad de destino: ");

    if (buscarCiudadPorId(origenId) == NULL ||
        buscarCiudadPorId(destinoId) == NULL) {
        cout << "ERROR: una o ambas ciudades no existen.\n";
        return;
    }

    if (!existeRuta(origenId, destinoId)) {
        cout << "ERROR: la ruta indicada no existe.\n";
        return;
    }

    quitarConexionEntre(origenId, destinoId);
    quitarConexionEntre(destinoId, origenId);

    cout << "Ruta eliminada correctamente.\n";
    registrarHistorial("Se elimino una ruta entre los ID " +
                       to_string(origenId) + " y " +
                       to_string(destinoId));
}

void consultarVecinos() {
    cout << "\n CONSULTAR VECINOS \n";

    int id = leerEntero("ID de la ciudad: ");
    Ciudad* ciudad = buscarCiudadPorId(id);

    if (ciudad == NULL) {
        cout << "ERROR: ciudad inexistente.\n";
        return;
    }

    cout << "Vecinos de " << ciudad->nombre << ":\n";

    Arista* arista = ciudad->adyacencias;

    if (arista == NULL) {
        cout << "No tiene ciudades vecinas.\n";
        return;
    }

    while (arista != NULL) {
        Ciudad* vecino = buscarCiudadPorId(arista->destinoId);

        if (vecino != NULL) {
            cout << "- " << vecino->nombre
                 << " | Distancia: " << arista->distancia << " km\n";
        }

        arista = arista->siguiente;
    }
}

void menuCiudades() {
    int opcion;

    do {
        cout << "\n MODULO DE CIUDADES \n";
        cout << "1. Registrar ciudad\n";
        cout << "2. Listar ciudades\n";
        cout << "3. Buscar ciudad y sus conexiones\n";
        cout << "4. Eliminar ciudad\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: registrarCiudad(); break;
            case 2: listarCiudades(); break;
            case 3: buscarCiudad(); break;
            case 4: eliminarCiudad(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}

void menuRutas() {
    int opcion;

    do {
        cout << "\n MODULO DE RUTAS \n";
        cout << "1. Crear ruta entre ciudades\n";
        cout << "2. Eliminar ruta\n";
        cout << "3. Consultar vecinos de una ciudad\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: crearRuta(); break;
            case 2: eliminarRuta(); break;
            case 3: consultarVecinos(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 5. MODULO DE PASAJEROS - LISTA ENLAZADA PROPIA


void insertarPasajero() {
    cout << "\n REGISTRAR PASAJERO \n";

    string nombre = leerTexto("Nombre completo: ");
    string telefono = leerTexto("Telefono o contacto: ");

    Pasajero* nuevo = new Pasajero;
    nuevo->id = siguienteIdPasajero++;
    nuevo->nombre = nombre;
    nuevo->telefono = telefono;
    nuevo->siguiente = NULL;

    if (listaPasajeros == NULL) {
        listaPasajeros = nuevo;
    } else {
        Pasajero* ultimo = listaPasajeros;

        while (ultimo->siguiente != NULL) {
            ultimo = ultimo->siguiente;
        }

        ultimo->siguiente = nuevo;
    }

    cout << "Pasajero registrado. ID asignado: " << nuevo->id << "\n";
    registrarHistorial("Se registro el pasajero: " + nombre);
}

void listarPasajeros() {
    cout << "\n LISTA DE PASAJEROS \n";

    if (listaPasajeros == NULL) {
        cout << "No hay pasajeros registrados.\n";
        return;
    }

    Pasajero* actual = listaPasajeros;

    while (actual != NULL) {
        cout << "ID: " << actual->id
             << " | Nombre: " << actual->nombre
             << " | Telefono: " << actual->telefono << "\n";

        actual = actual->siguiente;
    }
}

void buscarPasajero() {
    cout << "\n BUSCAR PASAJERO \n";

    int id = leerEntero("ID del pasajero: ");
    Pasajero* pasajero = buscarPasajeroPorId(id);

    if (pasajero == NULL) {
        cout << "No existe un pasajero con ese ID.\n";
        return;
    }

    cout << "Pasajero encontrado:\n";
    cout << "ID: " << pasajero->id << "\n";
    cout << "Nombre: " << pasajero->nombre << "\n";
    cout << "Telefono: " << pasajero->telefono << "\n";
}

void actualizarPasajero() {
    cout << "\n ACTUALIZAR PASAJERO \n";

    int id = leerEntero("ID del pasajero: ");
    Pasajero* pasajero = buscarPasajeroPorId(id);

    if (pasajero == NULL) {
        cout << "ERROR: pasajero inexistente.\n";
        return;
    }

    cout << "Deje el dato nuevo cuando se solicite.\n";
    pasajero->nombre = leerTexto("Nuevo nombre: ");
    pasajero->telefono = leerTexto("Nuevo telefono: ");

    cout << "Informacion actualizada correctamente.\n";
    registrarHistorial("Se actualizo el pasajero ID " + to_string(id));
}

bool pasajeroTieneSolicitudes(int id) {
    Solicitud* actual = frenteCola;

    while (actual != NULL) {
        if (actual->pasajeroId == id) {
            return true;
        }
        actual = actual->siguiente;
    }

    return false;
}

void eliminarPasajero() {
    cout << "\n ELIMINAR PASAJERO \n";

    int id = leerEntero("ID del pasajero: ");

    if (pasajeroTieneSolicitudes(id)) {
        cout << "No se puede eliminar: el pasajero tiene solicitudes pendientes.\n";
        return;
    }

    Pasajero* actual = listaPasajeros;
    Pasajero* anterior = NULL;

    while (actual != NULL && actual->id != id) {
        anterior = actual;
        actual = actual->siguiente;
    }

    if (actual == NULL) {
        cout << "ERROR: pasajero inexistente.\n";
        return;
    }

    string nombre = actual->nombre;

    if (anterior == NULL) {
        listaPasajeros = actual->siguiente;
    } else {
        anterior->siguiente = actual->siguiente;
    }

    delete actual;

    cout << "Pasajero eliminado correctamente.\n";
    registrarHistorial("Se elimino el pasajero: " + nombre);
}

void menuPasajeros() {
    int opcion;

    do {
        cout << "\n MODULO DE PASAJEROS \n";
        cout << "1. Registrar pasajero\n";
        cout << "2. Listar pasajeros\n";
        cout << "3. Buscar pasajero\n";
        cout << "4. Actualizar pasajero\n";
        cout << "5. Eliminar pasajero\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: insertarPasajero(); break;
            case 2: listarPasajeros(); break;
            case 3: buscarPasajero(); break;
            case 4: actualizarPasajero(); break;
            case 5: eliminarPasajero(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 6. MODULO DE COLA - COLA ENLAZADA PROPIA


void encolarSolicitud() {
    cout << "\n REGISTRAR SOLICITUD DE VIAJE \n";

    if (listaPasajeros == NULL) {
        cout << "Primero debe registrar al menos un pasajero.\n";
        return;
    }

    if (listaCiudades == NULL) {
        cout << "Primero debe registrar ciudades.\n";
        return;
    }

    listarPasajeros();
    int pasajeroId = leerEntero("ID del pasajero: ");

    if (buscarPasajeroPorId(pasajeroId) == NULL) {
        cout << "ERROR: el pasajero no existe.\n";
        return;
    }

    mostrarCiudadesEnConsola();

    int origenId = leerEntero("ID de ciudad de origen: ");
    int destinoId = leerEntero("ID de ciudad de destino: ");

    if (buscarCiudadPorId(origenId) == NULL ||
        buscarCiudadPorId(destinoId) == NULL) {
        cout << "ERROR: origen o destino inexistente.\n";
        return;
    }

    if (origenId == destinoId) {
        cout << "ERROR: origen y destino deben ser diferentes.\n";
        return;
    }

    Solicitud* nueva = new Solicitud;
    nueva->id = siguienteIdSolicitud++;
    nueva->pasajeroId = pasajeroId;
    nueva->origenId = origenId;
    nueva->destinoId = destinoId;
    nueva->siguiente = NULL;

    if (frenteCola == NULL) {
        frenteCola = nueva;
        finalCola = nueva;
    } else {
        finalCola->siguiente = nueva;
        finalCola = nueva;
    }

    cout << "Solicitud agregada a la cola. Numero: "
         << nueva->id << "\n";

    registrarHistorial("Se agrego solicitud de viaje " +
                       to_string(nueva->id));
}

void mostrarFrenteCola() {
    cout << "\n FRENTE DE LA COLA \n";

    if (frenteCola == NULL) {
        cout << "La cola esta vacia.\n";
        return;
    }

    Pasajero* pasajero = buscarPasajeroPorId(frenteCola->pasajeroId);
    Ciudad* origen = buscarCiudadPorId(frenteCola->origenId);
    Ciudad* destino = buscarCiudadPorId(frenteCola->destinoId);

    cout << "Solicitud: " << frenteCola->id << "\n";
    cout << "Pasajero: "
         << (pasajero != NULL ? pasajero->nombre : "No disponible") << "\n";
    cout << "Origen: "
         << (origen != NULL ? origen->nombre : "No disponible") << "\n";
    cout << "Destino: "
         << (destino != NULL ? destino->nombre : "No disponible") << "\n";
}

void listarCola() {
    cout << "\n SOLICITUDES PENDIENTES \n";

    if (frenteCola == NULL) {
        cout << "La cola esta vacia.\n";
        return;
    }

    Solicitud* actual = frenteCola;

    while (actual != NULL) {
        Pasajero* pasajero = buscarPasajeroPorId(actual->pasajeroId);
        Ciudad* origen = buscarCiudadPorId(actual->origenId);
        Ciudad* destino = buscarCiudadPorId(actual->destinoId);

        cout << "Solicitud " << actual->id
             << " | Pasajero: "
             << (pasajero != NULL ? pasajero->nombre : "No disponible")
             << " | Origen: "
             << (origen != NULL ? origen->nombre : "No disponible")
             << " | Destino: "
             << (destino != NULL ? destino->nombre : "No disponible")
             << "\n";

        actual = actual->siguiente;
    }
}

void atenderSolicitud() {
    cout << "\n ATENDER SOLICITUD \n";

    if (frenteCola == NULL) {
        cout << "No se puede atender: la cola esta vacia.\n";
        return;
    }

    Solicitud* atendida = frenteCola;

    Pasajero* pasajero = buscarPasajeroPorId(atendida->pasajeroId);
    string nombre = pasajero != NULL ? pasajero->nombre : "Pasajero";

    cout << "Se atendio la solicitud " << atendida->id
         << " del pasajero " << nombre << ".\n";

    frenteCola = frenteCola->siguiente;

    if (frenteCola == NULL) {
        finalCola = NULL;
    }

    registrarHistorial("Se atendio la solicitud " +
                       to_string(atendida->id));

    delete atendida;
}

void menuCola() {
    int opcion;

    do {
        cout << "\n COLA DE SOLICITUDES \n";
        cout << "1. Encolar solicitud de viaje\n";
        cout << "2. Atender primera solicitud\n";
        cout << "3. Mostrar frente de la cola\n";
        cout << "4. Listar solicitudes pendientes\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: encolarSolicitud(); break;
            case 2: atenderSolicitud(); break;
            case 3: mostrarFrenteCola(); break;
            case 4: listarCola(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 7. MODULO DE HISTORIAL


void mostrarHistorial() {
    cout << "\n HISTORIAL DE OPERACIONES \n";

    if (historial.empty()) {
        cout << "No hay operaciones registradas en esta sesion.\n";
        return;
    }

    for (size_t i = 0; i < historial.size(); i++) {
        cout << i + 1 << ". " << historial[i] << "\n";
    }
}

void menuHistorial() {
    int opcion;

    do {
        cout << "\n MODULO DE HISTORIAL \n";
        cout << "1. Mostrar historial\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: mostrarHistorial(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 8. MODULO DE BUSQUEDAS


void buscarRuta() {
    cout << "\n BUSCAR RUTA DIRECTA \n";

    int origenId = leerEntero("ID de origen: ");
    int destinoId = leerEntero("ID de destino: ");

    Ciudad* origen = buscarCiudadPorId(origenId);
    Ciudad* destino = buscarCiudadPorId(destinoId);

    if (origen == NULL || destino == NULL) {
        cout << "Una o ambas ciudades no existen.\n";
        return;
    }

    Arista* actual = origen->adyacencias;

    while (actual != NULL) {
        if (actual->destinoId == destinoId) {
            cout << "Ruta encontrada: " << origen->nombre
                 << " -> " << destino->nombre
                 << " | Distancia: " << actual->distancia << " km\n";
            return;
        }
        actual = actual->siguiente;
    }

    cout << "No existe una ruta directa entre esas ciudades.\n";
}

void menuBusquedas() {
    int opcion;

    do {
        cout << "\n MODULO DE BUSQUEDAS \n";
        cout << "1. Buscar ciudad por ID\n";
        cout << "2. Buscar pasajero por ID\n";
        cout << "3. Buscar ruta directa entre ciudades\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: buscarCiudad(); break;
            case 2: buscarPasajero(); break;
            case 3: buscarRuta(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 9. MODULO DE REPORTES


void generarReporte() {
    cout << "\n REPORTE GENERAL SITRAC \n";
    cout << "Cantidad de ciudades: " << contarCiudades() << "\n";
    cout << "Cantidad de rutas: " << contarRutas() << "\n";
    cout << "Cantidad de pasajeros: " << contarPasajeros() << "\n";
    cout << "Solicitudes pendientes: " << contarSolicitudes() << "\n";
    cout << "Operaciones en historial: " << historial.size() << "\n";
}

void menuReportes() {
    int opcion;

    do {
        cout << "\n MODULO DE REPORTES \n";
        cout << "1. Generar reporte general\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: generarReporte(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 10. MODULO DE ARCHIVOS: GUARDAR Y CARGAR


void guardarDatos() {
    ofstream archivo("sitrac_datos.txt");

    if (!archivo) {
        cout << "ERROR: no se pudo crear el archivo sitrac_datos.txt.\n";
        return;
    }

    // Ciudades
    archivo << "CIUDADES " << contarCiudades() << "\n";

    Ciudad* ciudad = listaCiudades;

    while (ciudad != NULL) {
        archivo << ciudad->id << " "
                << quoted(ciudad->nombre) << "\n";
        ciudad = ciudad->siguiente;
    }

    // Rutas: se guarda una sola vez cada conexion no dirigida.
    archivo << "RUTAS " << contarRutas() << "\n";

    ciudad = listaCiudades;

    while (ciudad != NULL) {
        Arista* arista = ciudad->adyacencias;

        while (arista != NULL) {
            if (ciudad->id < arista->destinoId) {
                archivo << ciudad->id << " "
                        << arista->destinoId << " "
                        << arista->distancia << "\n";
            }
            arista = arista->siguiente;
        }

        ciudad = ciudad->siguiente;
    }

    // Pasajeros
    archivo << "PASAJEROS " << contarPasajeros() << "\n";

    Pasajero* pasajero = listaPasajeros;

    while (pasajero != NULL) {
        archivo << pasajero->id << " "
                << quoted(pasajero->nombre) << " "
                << quoted(pasajero->telefono) << "\n";
        pasajero = pasajero->siguiente;
    }

    // Solicitudes pendientes
    archivo << "COLA " << contarSolicitudes() << "\n";

    Solicitud* solicitud = frenteCola;

    while (solicitud != NULL) {
        archivo << solicitud->id << " "
                << solicitud->pasajeroId << " "
                << solicitud->origenId << " "
                << solicitud->destinoId << "\n";
        solicitud = solicitud->siguiente;
    }

    // Historial
    archivo << "HISTORIAL " << historial.size() << "\n";

    for (size_t i = 0; i < historial.size(); i++) {
        archivo << quoted(historial[i]) << "\n";
    }

    archivo.close();

    cout << "Datos guardados correctamente en sitrac_datos.txt.\n";
    registrarHistorial("Se guardo la informacion del sistema");
}

void liberarTodaLaMemoria() {
    while (listaCiudades != NULL) {
        Ciudad* borrar = listaCiudades;
        listaCiudades = listaCiudades->siguiente;
        eliminarAristasDeCiudad(borrar);
        delete borrar;
    }

    while (listaPasajeros != NULL) {
        Pasajero* borrar = listaPasajeros;
        listaPasajeros = listaPasajeros->siguiente;
        delete borrar;
    }

    while (frenteCola != NULL) {
        Solicitud* borrar = frenteCola;
        frenteCola = frenteCola->siguiente;
        delete borrar;
    }

    finalCola = NULL;
}

void cargarDatos() {
    ifstream archivo("sitrac_datos.txt");

    if (!archivo) {
        cout << "No se encontro sitrac_datos.txt. No se modificaron los datos.\n";
        return;
    }

    string etiqueta;
    int cantidad;

    // Validar encabezado antes de limpiar los datos actuales.
    if (!(archivo >> etiqueta >> cantidad) ||
        etiqueta != "CIUDADES" || cantidad < 0) {
        cout << "ERROR: el archivo no tiene un formato SITRAC valido.\n";
        return;
    }

    liberarTodaLaMemoria();
    historial.clear();

    siguienteIdCiudad = 1;
    siguienteIdPasajero = 1;
    siguienteIdSolicitud = 1;

    int maxId = 0;

    for (int i = 0; i < cantidad; i++) {
        Ciudad* nueva = new Ciudad;
        nueva->adyacencias = NULL;
        nueva->siguiente = NULL;

        if (!(archivo >> nueva->id >> quoted(nueva->nombre))) {
            delete nueva;
            cout << "ERROR leyendo ciudades. Revise el archivo.\n";
            liberarTodaLaMemoria();
            return;
        }

        if (listaCiudades == NULL) {
            listaCiudades = nueva;
        } else {
            Ciudad* ultimo = listaCiudades;
            while (ultimo->siguiente != NULL) {
                ultimo = ultimo->siguiente;
            }
            ultimo->siguiente = nueva;
        }

        if (nueva->id > maxId) maxId = nueva->id;
    }

    siguienteIdCiudad = maxId + 1;

    if (!(archivo >> etiqueta >> cantidad) ||
        etiqueta != "RUTAS" || cantidad < 0) {
        cout << "ERROR leyendo la seccion de rutas.\n";
        liberarTodaLaMemoria();
        return;
    }

    for (int i = 0; i < cantidad; i++) {
        int origenId, destinoId;
        double distancia;

        if (!(archivo >> origenId >> destinoId >> distancia) ||
            distancia < 0) {
            cout << "ERROR leyendo una ruta.\n";
            liberarTodaLaMemoria();
            return;
        }

        Ciudad* origen = buscarCiudadPorId(origenId);
        Ciudad* destino = buscarCiudadPorId(destinoId);

        if (origen == NULL || destino == NULL) {
            cout << "ERROR: ruta con ciudad inexistente en el archivo.\n";
            liberarTodaLaMemoria();
            return;
        }

        insertarArista(origen, destinoId, distancia);
        insertarArista(destino, origenId, distancia);
    }

    if (!(archivo >> etiqueta >> cantidad) ||
        etiqueta != "PASAJEROS" || cantidad < 0) {
        cout << "ERROR leyendo pasajeros.\n";
        liberarTodaLaMemoria();
        return;
    }

    maxId = 0;

    for (int i = 0; i < cantidad; i++) {
        Pasajero* nuevo = new Pasajero;
        nuevo->siguiente = NULL;

        if (!(archivo >> nuevo->id
                      >> quoted(nuevo->nombre)
                      >> quoted(nuevo->telefono))) {
            delete nuevo;
            cout << "ERROR leyendo pasajero.\n";
            liberarTodaLaMemoria();
            return;
        }

        if (listaPasajeros == NULL) {
            listaPasajeros = nuevo;
        } else {
            Pasajero* ultimo = listaPasajeros;
            while (ultimo->siguiente != NULL) {
                ultimo = ultimo->siguiente;
            }
            ultimo->siguiente = nuevo;
        }

        if (nuevo->id > maxId) maxId = nuevo->id;
    }

    siguienteIdPasajero = maxId + 1;

    if (!(archivo >> etiqueta >> cantidad) ||
        etiqueta != "COLA" || cantidad < 0) {
        cout << "ERROR leyendo la cola.\n";
        liberarTodaLaMemoria();
        return;
    }

    maxId = 0;

    for (int i = 0; i < cantidad; i++) {
        Solicitud* nueva = new Solicitud;
        nueva->siguiente = NULL;

        if (!(archivo >> nueva->id
                      >> nueva->pasajeroId
                      >> nueva->origenId
                      >> nueva->destinoId)) {
            delete nueva;
            cout << "ERROR leyendo solicitud.\n";
            liberarTodaLaMemoria();
            return;
        }

        if (buscarPasajeroPorId(nueva->pasajeroId) == NULL ||
            buscarCiudadPorId(nueva->origenId) == NULL ||
            buscarCiudadPorId(nueva->destinoId) == NULL) {
            delete nueva;
            cout << "ERROR: solicitud con referencias inexistentes.\n";
            liberarTodaLaMemoria();
            return;
        }

        if (frenteCola == NULL) {
            frenteCola = nueva;
            finalCola = nueva;
        } else {
            finalCola->siguiente = nueva;
            finalCola = nueva;
        }

        if (nueva->id > maxId) maxId = nueva->id;
    }

    siguienteIdSolicitud = maxId + 1;

    if (!(archivo >> etiqueta >> cantidad) ||
        etiqueta != "HISTORIAL" || cantidad < 0) {
        cout << "ERROR leyendo historial.\n";
        liberarTodaLaMemoria();
        return;
    }

    for (int i = 0; i < cantidad; i++) {
        string evento;

        if (!(archivo >> quoted(evento))) {
            cout << "ERROR leyendo evento del historial.\n";
            liberarTodaLaMemoria();
            return;
        }

        historial.push_back(evento);
    }

    archivo.close();

    cout << "Informacion cargada correctamente.\n";
    registrarHistorial("Se cargo la informacion desde archivo");
}

void menuArchivos() {
    int opcion;

    do {
        cout << "\n MODULO DE ARCHIVOS \n";
        cout << "1. Guardar informacion\n";
        cout << "2. Cargar informacion\n";
        cout << "0. Volver al menu principal\n";

        opcion = leerEntero("Seleccione: ");

        switch (opcion) {
            case 1: guardarDatos(); break;
            case 2: cargarDatos(); break;
            case 0: cout << "Regresando al menu principal...\n"; break;
            default: cout << "Opcion no valida.\n";
        }
    } while (opcion != 0);
}


// 11. MENU PRINCIPAL

void mostrarMenuPrincipal() {
    cout << "                 SISTEMA SITRAC\n";
    cout << " Sistema Inteligente de Transporte y Gestion de Rutas\n";
    cout << "1. Ciudades\n";
    cout << "2. Rutas\n";
    cout << "3. Pasajeros\n";
    cout << "4. Cola de solicitudes de viaje\n";
    cout << "5. Historial\n";
    cout << "6. Busquedas\n";
    cout << "7. Reportes\n";
    cout << "8. Guardar / Cargar archivos\n";
    cout << "0. Salir\n";
}

int main() {
    int opcion;

    cout << "Bienvenido al Sistema SITRAC.\n";

    do {
        mostrarMenuPrincipal();
        opcion = leerEntero("Seleccione una opcion: ");

        switch (opcion) {
            case 1: menuCiudades(); break;
            case 2: menuRutas(); break;
            case 3: menuPasajeros(); break;
            case 4: menuCola(); break;
            case 5: menuHistorial(); break;
            case 6: menuBusquedas(); break;
            case 7: menuReportes(); break;
            case 8: menuArchivos(); break;

            case 0:
                cout << "Cerrando SITRAC. Hasta pronto.\n";
                break;

            default:
                cout << "ERROR: seleccione una opcion entre 0 y 8.\n";
        }

    } while (opcion != 0);

    liberarTodaLaMemoria();

    return 0;
}
