//# Reserva-de-Canchas

#include <iostream>
#include <vector>
#include <string>
#include <iomanip>

using namespace std;

// Estructura para almacenar los datos de una cancha
struct Cancha {
	int id;
	string tipo; // "Pádel" o "Tenis"
	double precioPorHora;
	bool disponible;
};

// Estructura para almacenar los datos de una reserva
struct Reserva {
	int idReserva;
	string nombreCliente;
	int idCancha;
	string horario;
};

// Clase principal que gestiona la lógica del sistema
class SistemaReservas {
private:
	vector<Cancha> canchas;
	vector<Reserva> reservas;
	int contadorReservas;
	
public:
	// Constructor: inicializa el inventario de canchas
	SistemaReservas() {
		contadorReservas = 1;
		canchas.push_back({1, "Pádel", 45000.0, true});
		canchas.push_back({2, "Pádel", 45000.0, true});
		canchas.push_back({3, "Tenis", 35000.0, true});
		canchas.push_back({4, "Tenis", 35000.0, true});
	}
	
	void mostrarCanchasDisponibles() {
		cout << "\n--- ESTADO DE LAS CANCHAS ---\n";
		cout << left << setw(5) << "ID" << setw(15) << "Tipo" 
			<< setw(15) << "Precio/Hora" << "Estado\n";
		cout << "--------------------------------------------------\n";
		for (const auto& c : canchas) {
			cout << left << setw(5) << c.id 
				<< setw(15) << c.tipo 
				<< "$" << setw(14) << c.precioPorHora;
			if (c.disponible) {
				cout << "Disponible\n";
			} else {
				cout << "Ocupada\n";
			}
		}
	}
	
	void realizarReserva() {
		string nombre, horario;
		int idCancha;
		
		mostrarCanchasDisponibles();
		
		cout << "\nIngrese el ID de la cancha que desea reservar: ";
		cin >> idCancha;
		
		bool canchaEncontrada = false;
		for (auto& c : canchas) {
			if (c.id == idCancha) {
				canchaEncontrada = true;
				if (!c.disponible) {
					cout << "Error: La cancha seleccionada ya está ocupada.\n";
					return;
				}
				
				cout << "Ingrese su nombre completo: ";
				cin.ignore();
				getline(cin, nombre);
				
				cout << "Ingrese el bloque horario (Ej: 14:00-15:00): ";
				getline(cin, horario);
				
				// Guardar la reserva en el vector
				reservas.push_back({contadorReservas, nombre, idCancha, horario});
				c.disponible = false; // Actualizar estado de la cancha
				
				cout << "\n¡Reserva registrada con éxito!\n";
				cout << "Su número de reserva es: " << contadorReservas << "\n";
				contadorReservas++;
				break;
			}
		}
		
		if (!canchaEncontrada) {
			cout << "Error: El ID ingresado no corresponde a ninguna cancha.\n";
		}
	}
	
	void mostrarReservas() {
		if (reservas.empty()) {
			cout << "\nNo hay reservas registradas en el sistema en este momento.\n";
			return;
		}
		
		cout << "\n--- LISTA DE RESERVAS ACTIVAS ---\n";
		cout << left << setw(12) << "ID Reserva" << setw(20) << "Cliente" 
			<< setw(12) << "ID Cancha" << "Horario\n";
		cout << "------------------------------------------------------------\n";
		for (const auto& r : reservas) {
			cout << left << setw(12) << r.idReserva 
				<< setw(20) << r.nombreCliente 
				<< setw(12) << r.idCancha 
				<< r.horario << "\n";
		}
	}
	
	void cancelarReserva() {
		if (reservas.empty()) {
			cout << "\nNo hay reservas activas para cancelar.\n";
			return;
		}
		
		int idReserva;
		cout << "\nIngrese el número de la reserva que desea cancelar: ";
		cin >> idReserva;
		
		bool encontrada = false;
		for (auto it = reservas.begin(); it != reservas.end(); ++it) {
			if (it->idReserva == idReserva) {
				encontrada = true;
				// Buscar la cancha asociada y liberarla
				for (auto& c : canchas) {
					if (c.id == it->idCancha) {
						c.disponible = true;
						break;
					}
				}
				// Borrar la reserva de la memoria
				reservas.erase(it);
				cout << "Reserva cancelada exitosamente. La cancha está libre de nuevo.\n";
				break;
			}
		}
		
		if (!encontrada) {
			cout << "Error: No se encontró ninguna reserva con ese número.\n";
		}
	}
};

void mostrarMenuPrincipal() {
	cout << "\n========================================\n";
	cout << "   CLUB DEPORTIVO - RESERVA DE CANCHAS    \n";
	cout << "========================================\n";
	cout << "1. Consultar disponibilidad de canchas\n";
	cout << "2. Realizar una nueva reserva\n";
	cout << "3. Ver historial de reservas\n";
	cout << "4. Cancelar una reserva existente\n";
	cout << "5. Salir de la aplicación\n";
	cout << "========================================\n";
	cout << "Ingrese una opción (1-5): ";
}

int main() {
	SistemaReservas sistema;
	int opcion;
	
	do {
		mostrarMenuPrincipal();
		// Validación básica de entrada
		if (!(cin >> opcion)) {
			cin.clear();
			cin.ignore(10000, '\n');
			opcion = 0;
		}
		
		switch (opcion) {
		case 1:
			sistema.mostrarCanchasDisponibles();
			break;
		case 2:
			sistema.realizarReserva();
			break;
		case 3:
			sistema.mostrarReservas();
			break;
		case 4:
			sistema.cancelarReserva();
			break;
		case 5:
			cout << "\nCerrando el sistema. ¡Hasta pronto!\n";
			break;
		default:
			cout << "\nOpción inválida. Por favor, seleccione un número del 1 al 5.\n";
		}
	} while (opcion != 5);
	
	return 0;
}
