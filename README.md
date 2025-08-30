# PRIMER-PROYECTO
Desarrolla una aplicación de consola que permita al Comité Olímpico Guatemalteco registrar y analizar el rendimiento de sus atletas de alto nivel.
<p align="center">
<img width="481" height="551" alt="DiagramaUML" src="https://github.com/user-attachments/assets/cfde8e83-c9a4-462e-88bd-1815a2a20051" />
</p>

```java
import java.io.*;
import java.util.*;

class Entrenamiento {
    String fecha;
    String tipo;
    double valor;

    Entrenamiento(String fecha, String tipo, double valor) {
        this.fecha = fecha;
        this.tipo = tipo;
        this.valor = valor;
    }
}

class Atleta {
    String nombre, disciplina, departamento;
    int edad;
    List<Entrenamiento> entrenamientos = new ArrayList<>();

    Atleta(String nombre, int edad, String disciplina, String departamento) {
        this.nombre = nombre;
        this.edad = edad;
        this.disciplina = disciplina;
        this.departamento = departamento;
    }

    void agregarEntrenamiento(Entrenamiento e) {
        entrenamientos.add(e);
    }

    double promedio() {
        return entrenamientos.stream().mapToDouble(e -> e.valor).average().orElse(0);
    }

    double mejorMarca() {
        return entrenamientos.stream().mapToDouble(e -> e.valor).max().orElse(0);
    }
}

public class Rendimiento {
    static List<Atleta> atletas = new ArrayList<>();
    static final String ARCHIVO = "atletas.csv";

    public static void main(String[] args) throws Exception {
        cargar();
        Scanner sc = new Scanner(System.in);

        while (true) {
            System.out.println("\n--- Comité Olímpico Guatemalteco ---");
            System.out.println("1. Registrar atleta");
            System.out.println("2. Registrar entrenamiento");
            System.out.println("3. Mostrar historial");
            System.out.println("4. Estadísticas");
            System.out.println("5. Eliminar datos");
            System.out.println("6. Guardar y salir");

            String op = sc.nextLine();
            switch (op) {
                case "1":
                    System.out.print("Nombre: "); String n = sc.nextLine();
                    System.out.print("Edad: "); int e = Integer.parseInt(sc.nextLine());
                    System.out.print("Disciplina: "); String d = sc.nextLine();
                    System.out.print("Departamento: "); String dep = sc.nextLine();
                    atletas.add(new Atleta(n,e,d,dep));
                    break;

                case "2":
                    System.out.print("Nombre del atleta: "); n = sc.nextLine();
                    Atleta a = buscar(n);
                    if (a != null) {
                        System.out.print("Fecha (YYYY-MM-DD): "); String f = sc.nextLine();
                        System.out.print("Tipo: "); String t = sc.nextLine();
                        System.out.print("Valor: "); double v = Double.parseDouble(sc.nextLine());
                        a.agregarEntrenamiento(new Entrenamiento(f,t,v));
                    } else {
                        System.out.println("⚠️ No existe el atleta ingresado.");
                    }
                    break;

                case "3": // HISTORIAL
                    System.out.print("Nombre: "); n = sc.nextLine();
                    a = buscar(n);
                    if (a != null) {
                        if (a.entrenamientos.isEmpty()) {
                            System.out.println("⚠️ Este atleta no tiene entrenamientos.");
                        } else {
                            System.out.println("\nHistorial de " + a.nombre + ":");
                            for (int i=0; i<a.entrenamientos.size(); i++) {
                                Entrenamiento en = a.entrenamientos.get(i);
                                System.out.println((i+1)+". "+en.fecha+" | "+en.tipo+" | "+en.valor);
                            }
                        }
                    } else {
                        System.out.println("⚠️ No existe el atleta ingresado.");
                    }
                    break;

                case "4": // ESTADÍSTICAS
                    System.out.print("Nombre: "); n = sc.nextLine();
                    a = buscar(n);
                    if (a != null) {
                        if (!a.entrenamientos.isEmpty()) {
                            System.out.println("\nEstadísticas de " + a.nombre + ":");
                            System.out.println("Promedio: " + a.promedio());
                            System.out.println("Mejor marca: " + a.mejorMarca());
                        } else {
                            System.out.println("⚠️ Este atleta no tiene entrenamientos.");
                        }
                    } else {
                        System.out.println("⚠️ No existe el atleta ingresado.");
                    }
                    break;

                case "5": // ELIMINAR DATOS
                    System.out.println("1. Eliminar atleta");
                    System.out.println("2. Eliminar entrenamiento");
                    String sub = sc.nextLine();

                    if (sub.equals("1")) {
                        System.out.print("Nombre del atleta a eliminar: "); n = sc.nextLine();
                        a = buscar(n);
                        if (a != null) {
                            atletas.remove(a);
                            System.out.println("✅ Atleta eliminado.");
                        } else {
                            System.out.println("⚠️ No existe el atleta ingresado.");
                        }
                    } else if (sub.equals("2")) {
                        System.out.print("Nombre del atleta: "); n = sc.nextLine();
                        a = buscar(n);
                        if (a != null) {
                            if (!a.entrenamientos.isEmpty()) {
                                for (int i=0; i<a.entrenamientos.size(); i++) {
                                    Entrenamiento en = a.entrenamientos.get(i);
                                    System.out.println((i+1)+". "+en.fecha+" | "+en.tipo+" | "+en.valor);
                                }
                                System.out.print("Número de entrenamiento a eliminar: ");
                                int idx = Integer.parseInt(sc.nextLine()) - 1;
                                if (idx >= 0 && idx < a.entrenamientos.size()) {
                                    a.entrenamientos.remove(idx);
                                    System.out.println("✅ Entrenamiento eliminado.");
                                } else {
                                    System.out.println("⚠️ Número inválido.");
                                }
                            } else {
                                System.out.println("⚠️ Este atleta no tiene entrenamientos.");
                            }
                        } else {
                            System.out.println("⚠️ No existe el atleta ingresado.");
                        }
                    }
                    break;

                case "6":
                    guardar();
                    System.out.println("💾 Datos guardados. 👋");
                    return;
            }
        }
    }

    static Atleta buscar(String nombre) {
        for (Atleta a : atletas)
            if (a.nombre.equalsIgnoreCase(nombre)) return a;
        return null;
    }

    static void guardar() throws IOException {
        PrintWriter pw = new PrintWriter(new FileWriter(ARCHIVO));
        for (Atleta a : atletas)
            for (Entrenamiento e : a.entrenamientos)
                pw.println(a.nombre+","+a.edad+","+a.disciplina+","+a.departamento+","+
                        e.fecha+","+e.tipo+","+e.valor);
        pw.close();
    }

    static void cargar() {
        try (BufferedReader br = new BufferedReader(new FileReader(ARCHIVO))) {
            String linea;
            while ((linea = br.readLine()) != null) {
                String[] p = linea.split(",");
                Atleta a = buscar(p[0]);
                if (a == null) {
                    a = new Atleta(p[0], Integer.parseInt(p[1]), p[2], p[3]);
                    atletas.add(a);
                }
                a.agregarEntrenamiento(new Entrenamiento(p[4], p[5], Double.parseDouble(p[6])));
            }
        } catch (IOException ignored) {}
    }
}
