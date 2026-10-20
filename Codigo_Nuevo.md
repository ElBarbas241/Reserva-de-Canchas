#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <iomanip>

using namespace std;

// ============================================================================
// 1. PATRÓN CREACIONAL: FACTORY METHOD
// ============================================================================
class Cancha {
protected:
	int id;
	string tipo;
	double precioPorHora;
	bool disponible = true;
public:
	Cancha(int i, string t, double p) : id(i), tipo(t), precioPorHora(p) {}
	virtual ~Cancha() = default;
	int getId() const { return id; }
	double getPrecio() const { return precioPorHora; }
	bool isDisponible() const { return disponible; }
	void setDisponible(bool d) { disponible = d; }
	void mostrar() const {
		cout << left << setw(5) << id << setw(15) << tipo << "$" << setw(14) << precioPorHora << (disponible ? "Disponible\n" : "Ocupada\n");
	}
};

class CanchaFutbol : public Cancha { public: CanchaFutbol(int i) : Cancha(i, "Futbol 5", 50.0) {} };
class CanchaTenis : public Cancha { public: CanchaTenis(int i) : Cancha(i, "Tenis", 35.0) {} };
class CanchaPadel : public Cancha { public: CanchaPadel(int i) : Cancha(i, "Padel", 40.0) {} };

class CanchaFactory {
public:
	static unique_ptr<Cancha> crearCancha(int id, int tipo) {
		if (tipo == 1) return unique_ptr<Cancha>(new CanchaFutbol(id));
		if (tipo == 2) return unique_ptr<Cancha>(new CanchaTenis(id));
		if (tipo == 3) return unique_ptr<Cancha>(new CanchaPadel(id));
		return nullptr;
	}
};

// ============================================================================
// 2. PATRÓN ESTRUCTURAL: DECORATOR
// ============================================================================
class ServicioReserva {
public:
	virtual ~ServicioReserva() = default;
	virtual string getDetalle() const = 0;
	virtual double getCosto() const = 0;
};

class ReservaBase : public ServicioReserva {
	string cliente, horario;
	double precioBase;
public:
	ReservaBase(string c, string h, double p) : cliente(c), horario(h), precioBase(p) {}
	string getDetalle() const override { return "Base (" + horario + ")"; }
	double getCosto() const override { return precioBase; }
};

class DecoradorServicio : public ServicioReserva {
protected:
	shared_ptr<ServicioReserva> envoltorio;
public:
	DecoradorServicio(shared_ptr<ServicioReserva> s) : envoltorio(s) {}
};

class DecoradorIluminacion : public DecoradorServicio {
public:
	DecoradorIluminacion(shared_ptr<ServicioReserva> s) : DecoradorServicio(s) {}
	string getDetalle() const override { return envoltorio->getDetalle() + " + Luces"; }
	double getCosto() const override { return envoltorio->getCosto() + 15.0; }
};

class DecoradorEquipamiento : public DecoradorServicio {
public:
	DecoradorEquipamiento(shared_ptr<ServicioReserva> s) : DecoradorServicio(s) {}
	string getDetalle() const override { return envoltorio->getDetalle() + " + Equipamiento"; }
	double getCosto() const override { return envoltorio->getCosto() + 10.0; }
};

// ============================================================================
// 3. PATRÓN DE COMPORTAMIENTO: OBSERVER
// ============================================================================
class IObservador {
public:
	virtual ~IObservador() = default;
	virtual void notificar(const string& msg) = 0;
};

class NotificadorPantalla : public IObservador {
public:
	void notificar(const string& msg) override { cout << "[NOTIFICACION]: " << msg << "\n"; }
};

struct ReservaRegistro { int idReserva; string cliente; int idCancha; string detalles; double costo; string horario; };

class GestorCanchas {
	vector<unique_ptr<Cancha>> canchas;
	vector<ReservaRegistro> reservas;
	vector<shared_ptr<IObservador>> observadores;
	int contador = 101;
	
	void notificar(const string& msg) { for (auto& o : observadores) o->notificar(msg); }
public:
		void agregarObservador(shared_ptr<IObservador> o) { observadores.push_back(o); }
		void agregarCancha(int id, int tipo) { if (auto c = CanchaFactory::crearCancha(id, tipo)) canchas.push_back(move(c)); }
		
		void mostrarCanchas() const {
			cout << "\n--- ESTADO DE LAS CANCHAS ---\n";
			cout << left << setw(5) << "ID" << setw(15) << "Tipo" << setw(15) << "Precio/Hora" << "Estado\n";
			cout << "--------------------------------------------------\n";
			for (const auto& c : canchas) c->mostrar();
		}
		
		void realizarReserva() {
			mostrarCanchas();
			int id; cout << "\nIngrese el ID de la cancha que desea reservar: "; cin >> id;
			for (auto& c : canchas) {
				if (c->getId() == id) {
					if (!c->isDisponible()) {
						cout << "Error: La cancha seleccionada ya esta ocupada.\n";
						return;
					}
					string nom, hor; char opc;
					cout << "Ingrese su nombre completo: "; cin.ignore(); getline(cin, nom);
					cout << "Ingrese el bloque horario (Ej: 14:00-15:00): "; getline(cin, hor);
					
					shared_ptr<ServicioReserva> s = make_shared<ReservaBase>(nom, hor, c->getPrecio());
					cout << "¿Desea incluir iluminacion nocturna? (+$15) [s/n]: "; cin >> opc;
					if (opc == 's' || opc == 'S') s = make_shared<DecoradorIluminacion>(s);
					cout << "¿Desea incluir alquiler de equipamiento? (+$10) [s/n]: "; cin >> opc;
					if (opc == 's' || opc == 'S') s = make_shared<DecoradorEquipamiento>(s);
					
					c->setDisponible(false);
					reservas.push_back({contador, nom, id, s->getDetalle(), s->getCosto(), hor});
					cout << "\n¡Reserva registrada con exito!\n";
					cout << "Su numero de reserva es: " << contador << "\n";
					cout << "Costo Total: $" << s->getCosto() << "\n";
					
					notificar("Reserva #" + to_string(contador) + " creada por " + nom + " para la Cancha " + to_string(id));
					contador++;
					return;
				}
			}
			cout << "Error: El ID ingresado no corresponde a ninguna cancha.\n";
		}
		
		void mostrarReservas() const {
			if (reservas.empty()) {
				cout << "\nNo hay reservas registradas en el sistema en este momento.\n";
				return;
			}
			cout << "\n--- LISTA DE RESERVAS ACTIVAS ---\n";
			cout << left << setw(12) << "ID Reserva" << setw(18) << "Cliente" << setw(10) << "ID Cancha" << "Horario\n";
			cout << "------------------------------------------------------------\n";
			for (const auto& r : reservas) {
				cout << left << setw(12) << r.idReserva << setw(18) << r.cliente << setw(10) << r.idCancha << r.horario << "\n";
			}
		}
		
		void cancelarReserva() {
			if (reservas.empty()) {
				cout << "\nNo hay reservas activas para cancelar.\n";
				return;
			}
			int idR; cout << "\nIngrese el numero de la reserva que desea cancelar: "; cin >> idR;
			for (auto it = reservas.begin(); it != reservas.end(); ++it) {
				if (it->idReserva == idR) {
					for (auto& c : canchas) if (c->getId() == it->idCancha) c->setDisponible(true);
					reservas.erase(it);
					cout << "Reserva cancelada exitosamente. La cancha esta libre de nuevo.\n";
					notificar("Reserva #" + to_string(idR) + " cancelada.");
					return;
				}
			}
			cout << "Error: No se encontro ninguna reserva con ese numero.\n";
		}
};

int main() {
	GestorCanchas gestor;
	gestor.agregarObservador(make_shared<NotificadorPantalla>());
	gestor.agregarCancha(1, 1); gestor.agregarCancha(2, 2); gestor.agregarCancha(3, 3);
	
	int opc = 0;
	do {
		cout << "\n=========================================\n";
		cout << "       SISTEMA DE RESERVA DE CANCHAS     \n";
		cout << "=========================================\n";
		cout << "1. Ver estado de canchas\n";
		cout << "2. Realizar reserva\n";
		cout << "3. Ver reservas activas\n";
		cout << "4. Cancelar reserva\n";
		cout << "5. Salir\n";
		cout << "Elija una opcion: ";
		cin >> opc;
		
		switch (opc) {
		case 1: gestor.mostrarCanchas(); break;
		case 2: gestor.realizarReserva(); break;
		case 3: gestor.mostrarReservas(); break;
		case 4: gestor.cancelarReserva(); break;
		case 5: cout << "Saliendo del sistema...\n"; break;
		default: cout << "Opcion no valida. Intente de nuevo.\n";
		}
	} while (opc != 5);
	return 0;
}
