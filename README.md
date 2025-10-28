# Prueba-Corta-I-De-la-idea-al-navegador-tu-CRUD-toma-vida-

CREATE DATABASE crud_estudiantes;
USE crud_estudiantes;

CREATE TABLE estudiantes (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  correo VARCHAR(100),
  carrera VARCHAR(100),
  imagen VARCHAR(255)
);

package modelo;

import java.sql.Connection;
import java.sql.DriverManager;

public class Conexion {
    Connection con;
    public Connection getConnection(){
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            con = DriverManager.getConnection("jdbc:mysql://localhost:3306/crud_estudiantes", "root", ""); 
        } catch (Exception e) {
            System.out.println("Error de conexión: " + e);
        }
        return con;
    }
}


package modelo;

public class Estudiante {
    private int id;
    private String nombre;
    private String correo;
    private String carrera;
    private String imagen;

    public Estudiante() {}

    public Estudiante(int id, String nombre, String correo, String carrera, String imagen) {
        this.id = id;
        this.nombre = nombre;
        this.correo = correo;
        this.carrera = carrera;
        this.imagen = imagen;
    }

    // Getters y Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public String getCorreo() { return correo; }
    public void setCorreo(String correo) { this.correo = correo; }

    public String getCarrera() { return carrera; }
    public void setCarrera(String carrera) { this.carrera = carrera; }

    public String getImagen() { return imagen; }
    public void setImagen(String imagen) { this.imagen = imagen; }
}

package modelo;

import java.sql.*;
import java.util.*;

public class EstudianteDAO {
    Connection con;
    Conexion cn = new Conexion();
    PreparedStatement ps;
    ResultSet rs;

    public List<Estudiante> listar() {
        List<Estudiante> lista = new ArrayList<>();
        String sql = "SELECT * FROM estudiantes";
        try {
            con = cn.getConnection();
            ps = con.prepareStatement(sql);
            rs = ps.executeQuery();
            while (rs.next()) {
                Estudiante e = new Estudiante();
                e.setId(rs.getInt("id"));
                e.setNombre(rs.getString("nombre"));
                e.setCorreo(rs.getString("correo"));
                e.setCarrera(rs.getString("carrera"));
                e.setImagen(rs.getString("imagen"));
                lista.add(e);
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return lista;
    }

    public void agregar(Estudiante e) {
        String sql = "INSERT INTO estudiantes(nombre, correo, carrera, imagen) VALUES(?,?,?,?)";
        try {
            con = cn.getConnection();
            ps = con.prepareStatement(sql);
            ps.setString(1, e.getNombre());
            ps.setString(2, e.getCorreo());
            ps.setString(3, e.getCarrera());
            ps.setString(4, e.getImagen());
            ps.executeUpdate();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public void eliminar(int id) {
        String sql = "DELETE FROM estudiantes WHERE id=?";
        try {
            con = cn.getConnection();
            ps = con.prepareStatement(sql);
            ps.setInt(1, id);
            ps.executeUpdate();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}

<%@page import="modelo.*"%>
<%@page import="java.util.*"%>
<%@page contentType="text/html" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>CRUD Estudiantes</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container mt-5">
        <h1 class="text-center mb-4">Gestión de Estudiantes</h1>

        <!-- Formulario de registro -->
        <form action="agregar.jsp" method="post" class="card p-3 mb-4 shadow-sm">
            <div class="row">
                <div class="col-md-3"><input type="text" name="nombre" placeholder="Nombre" class="form-control" required></div>
                <div class="col-md-3"><input type="email" name="correo" placeholder="Correo" class="form-control" required></div>
                <div class="col-md-3"><input type="text" name="carrera" placeholder="Carrera" class="form-control" required></div>
                <div class="col-md-3"><input type="text" name="imagen" placeholder="URL de imagen" class="form-control"></div>
            </div>
            <div class="text-center mt-3">
                <button class="btn btn-success">Agregar</button>
            </div>
        </form>

        <!-- Cards de estudiantes -->
        <div class="row">
            <%
                EstudianteDAO dao = new EstudianteDAO();
                List<Estudiante> lista = dao.listar();
                for(Estudiante e : lista){
            %>
            <div class="col-md-3 mb-4">
                <div class="card shadow-sm">
                    <img src="<%= e.getImagen() %>" class="card-img-top" alt="Imagen" style="height:200px; object-fit:cover;">
                    <div class="card-body">
                        <h5 class="card-title"><%= e.getNombre() %></h5>
                        <p class="card-text"><%= e.getCorreo() %><br><%= e.getCarrera() %></p>
                        <a href="eliminar.jsp?id=<%= e.getId() %>" class="btn btn-danger btn-sm">Eliminar</a>
                    </div>
                </div>
            </div>
            <% } %>
        </div>
    </div>
</body>
</html>

<%@page import="modelo.*"%>
<%
    Estudiante e = new Estudiante();
    e.setNombre(request.getParameter("nombre"));
    e.setCorreo(request.getParameter("correo"));
    e.setCarrera(request.getParameter("carrera"));
    e.setImagen(request.getParameter("imagen"));

    EstudianteDAO dao = new EstudianteDAO();
    dao.agregar(e);
    response.sendRedirect("index.jsp");
%>

<%@page import="modelo.*"%>
<%
    int id = Integer.parseInt(request.getParameter("id"));
    EstudianteDAO dao = new EstudianteDAO();
    dao.eliminar(id);
    response.sendRedirect("index.jsp");
%>
