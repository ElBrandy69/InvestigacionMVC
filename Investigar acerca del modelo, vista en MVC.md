
#Modelo MVC

##¿Que es el modelo mvc?
Modelo Vista Controlador (MVC) es un estilo de arquitectura de software que separa los datos de una aplicación, la interfaz de usuario, y la lógica de control en tres componentes distintos.

###Describir el modelo de representación de los datos.
Se trata de un modelo muy maduro y que ha demostrado su validez a lo largo de los años en todo tipo de aplicaciones, y sobre multitud de lenguajes y plataformas de desarrollo.

##Modelo
El Modelo que contiene una representación de los datos que maneja el sistema, su lógica de negocio, y sus mecanismos de persistencia.

Ejemplo
~~~
<?php

require 'bd/conexion_bd.php';

class Productos extends BD_PDO
{
	function Insertar($nombre, $cantidad, $idproveedor, $idcategoria)
	{
		$this->Ejecutar_Instruccion("Insert into productos(Nombre,Cantidad,id_proveedor,id_categoria) values('$nombre','$cantidad','$idproveedor','$idcategoria')");
	}

	function Eliminar($id)
	{
		$this->Ejecutar_Instruccion("Delete from productos where id_producto = '$id'");
	}

	function Buscar($buscar)
	{
		$datos_buscar = $this->Ejecutar_Instruccion("Select productos.id_producto,productos.Nombre,productos.Cantidad,concat(proveedores.Nombres,' ',proveedores.Apellido_p,' ',proveedores.Apellido_m) as Nombre_prov,id_categoria from productos INNER JOIN proveedores ON productos.id_proveedor=proveedores.id_proveedor where Nombre like '%$buscar%'");
		return $datos_buscar;
	}

	function Buscar_todo()
	{

		$datos_buscar = $this ->Ejecutar_Instruccion("Select productos.id_producto,productos.Nombre,productos.Cantidad,concat(proveedores.Nombres,' ',proveedores.Apellido_p,' ',proveedores.Apellido_m) as Nombre_prov,categorias.Nombre from productos INNER JOIN proveedores ON productos.id_proveedor=proveedores.id_proveedor INNER JOIN categorias ON productos.id_categoria=categorias.id_categoria;");
		return $datos_buscar;
	
	}

	function Modificar($nombre,$cantidad,$idproveedor,$idcategoria,$id_producto)
	{	
		
		$this->Ejecutar_Instruccion("Update productos set Nombre='$nombre',Cantidad='$cantidad',id_proveedor='$idproveedor',id_categoria='$idcategoria' where id_producto = '$id_producto'");
	}

	function Seleccionar($id)
	{
		$prod_mod = $this->Ejecutar_Instruccion("Select * from productos where id_producto = '$id'");
		return	$prod_mod;

	}

	function Tabla_gen($datos_buscar)
	{
		$tabla="";
		foreach ($datos_buscar as $renglon) {
			$tabla.="<tr>";
			$tabla.='<td>'.$renglon[0].'</td>';
			$tabla.='<td>'.$renglon[1].'</td>';
			$tabla.='<td>'.$renglon[2].'</td>';
			$tabla.='<td>'.$renglon[3].'</td>';
			$tabla.='<td>'.$renglon[4].'</td>';

			if ($_SESSION['privilegio']=='Admin') 
			{

				$tabla.='<td><input class="btn-danger" type="button" id="btneliminar" name="btneliminar" value="Eliminar" onclick="javascript: eliminar('.$renglon[0].');"></td>';
				$tabla.='<td><input class="btn-warning" type="button" id="btnmodificar" name="btnmodificar" value="Modificar" onclick="javascript: modificar('.$renglon[0].');"></td>
				</tr>';

			}	
		}
			return $tabla;
	}
}

?>
~~~

##Vista
La Vista, o interfaz de usuario, que compone la información que se envía al cliente y los mecanismos interacción con éste.
Ejemplo: 

~~~
<?php 

error_reporting(1);
session_start();

 ?>

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Productos</title>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<link href="vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
<script src="vendor/jquery/jquery.slim.min.js"></script>
  <script src="vendor/bootstrap/js/bootstrap.bundle.min.js"></script>

<script>
	function eliminar(id)
	{
		if (confirm("¿ Estas seguro de eliminar el registro ?")) 
		{
			window.location = "index.php?ideliminar=" + id;
		}
	}

	function modificar(id)
	{
		window.location = "index.php?idmodificar=" + id;
	}

	function cerrar_sesion()
	{
		if (confirm("¿ Estas seguro de cerrar la sesión ?")) 
		{
			window.location = "cerrar_sesion.php";
		} 	
	}

	function validar()
	{
		var nombre = document.getElementById("txtnombre").value;
		var id = document.getElementById("lstcategorias").value;

		if (nombre.trim().length<1) 
		{
			alert("Nombre esta vacio");
			return false;
		}

		if (nombre.trim().length != nombre.length) 
		{
			alert("Tienes espacios de mas en el nombre");
			return false;
		}

		$.getJSON("validaciones/verificar_categorias.php?c=" + id).done(function(datos)  
	    {
	      if ((isset(datos[0][0]))) 
	      {
	        alert("Categoria no existe, No te quieras pasar de listo");
	        return false;
	      }        
	    }); 

		return true;
	}

	function verificar_producto(id)
	{
	  $.getJSON("validaciones/verificar_producto.php?p=" + id).done(function(datos)  
	    {
	      if (datos[0][0]>0) 
	      {
	        alert("Producto ya existe, verifique");
	      }        
	    });  
	}


</script>
<style>
	body{
		background-color:black;
	}

	h1{
		color: white;
	}

	select{
		width: 208px;
		height: 30px;
		border-radius: 0;
	}
</style>
</head>
<body>
	<div class="container">
	<br>	
	<h1>Productos</h1>
	
	<br>
	<div class="container" style="display: flex;justify-content:center;">
	<form action="index.php" method="post" id="frminsertar" name="frminsertar" 
	onsubmit="return validar();">
		<input type="text" id="txtid" name="txtid" placeholder="Numero" value="<?php echo @$prod_mod[0][0]; ?>" hidden>
		<input type="text" id="txtnombre" name="txtnombre" onblur="javascript: verificar_producto(this.value);" maxlength="30" placeholder="Nombre" value="<?php echo @$prod_mod[0][1]; ?>" required>
		<input type="text" id="txtcantidad" name="txtcantidad" maxlength="5" placeholder="Cantidad" value="<?php echo @$prod_mod[0][2]; ?>" required>
		<select name="lstproveedores" id="lstproveedores" required>
			<option value="">Seleccione Proveedor</option>
			<?php echo $datos_proveedores; ?>
		</select>

		<select name="lstcategorias" id="lstcategorias" required>
			<option value="">Seleccione Categorias</option>
			<?php echo $datos_categorias; ?>
		</select>

		<input class="btn-info" type="submit" id="btnregistrar" name="btnregistrar" value="<?php 
		if(isset($_GET['idmodificar']))
		{
			echo 'Guardar';
		}
		else
		{
			echo 'Insertar';
		}
		?>">		
	</form>	
	</div>
	
	<br>
	<ul>
		<div class="elemento1" style="float: inline-start;">
			<h1>Listado de Productos</h1>
				<form action="index.php" id="frmbuscar" name="frmbuscar" method="post">
					<input type="text" id="txtbuscar" name="txtbuscar" placeholder="Buscar nombre">
					<input class="btn-primary" type="submit" id="btnbuscar" name="btnbuscar" value="Buscar">
				</form>
		</div>
		<div class="elemento2" style="float: inline-end;">
			<h1>Sesiones</h1>
			<a href="iniciar_sesion.php">Iniciar Sesion</a>
			<input class="btn-success" type="button" id="btncerrarsesion" name="btncerrarsesion" value="Cerrar Sesion" onclick="javascript: cerrar_sesion();">
		</div>
		<br>
	</ul>
	<br>
		<table class="table table-striped table-dark" align="center" style="margin-top: 5%;">
			<tr>
				<td>Num</td>
				<td>Nombre</td>
				<td>Cantidad</td>
				<td>Proveedor</td>
				<td>Categoria</td>
				<td colspan="2" align="center">Accion</td>
			</tr>
			<?php echo $datos; ?>
		</table>

<br><br>

</div>
	
</body>
</html>

~~~

##Controlador
El Controlador, que actúa como intermediario entre el Modelo y la Vista, gestionando el flujo de información entre ellos y las transformaciones para adaptar los datos a las necesidades de cada uno.
Ejemplo: 

~~~
<?php 
error_reporting(1);
session_start();

 ?>

	<?php 

	require 'modelo.php';

	$obj = new Productos();

	

	if (isset($_POST['btnregistrar'])) 
	{
		$nombre = $_POST['txtnombre'];
		$cantidad = $_POST['txtcantidad'];
		$idproveedor = $_POST['lstproveedores'];
		$idcategoria = $_POST['lstcategorias'];

		if ($_POST['btnregistrar']=='Insertar') 
		{
			
			$obj->Insertar($nombre,$cantidad,$idproveedor,$idcategoria);
			header('Location:index.php');
			
		}
		else if ($_POST['btnregistrar']=='Guardar')
		{
			$id_producto=$_POST['txtid'];
			$obj->Modificar($nombre,$cantidad,$idproveedor,$idcategoria,$id_producto);
			header('Location:index.php');
		}
		
	}
	else if(isset($_GET['ideliminar']))
	{
		$id = $_GET['ideliminar'];
		$obj->Eliminar($id);
	}


	else if(isset($_GET['idmodificar']))
	{
		$id = $_GET['idmodificar'];
		$prod_mod = $obj->Seleccionar($id);
	}

	if (isset($_POST['btnbuscar'])) 
	{
		$buscar = $_POST['txtbuscar'];
		$result = $obj->Buscar($buscar);
		$datos = $obj->Tabla_gen($result);
	}
	else{
		$result = $obj->Buscar_todo();
		$datos = $obj->Tabla_gen($result);
	}


		$buscar = $_POST['txtbuscar']; 

		$datos_buscar = $obj->Buscar($buscar);

		$datos_proveedores = $obj->listados("Select id_proveedor,concat(Nombres,' ',Apellido_p,' ',Apellido_m) as Nombre_comp from proveedores","Select id_proveedor from productos");
		$datos_categorias = $obj->listados("Select id_categoria,Nombre from categorias","Select id_categoria from productos");


		require 'vista.php';
	 ?>


	
~~~

##Enlistar las funcionalidades de la aplicación.
A continuación se presentan las funcionalidades principales del modelo MVC:

- Modelo: El modelo es la capa de datos de la aplicación, es decir, donde se encuentran los datos y la lógica de negocio. Esta capa es responsable de interactuar con la base de datos, realizar operaciones de lectura y escritura de datos, y llevar a cabo el procesamiento de datos.

- Vista: La vista es la capa de presentación de la aplicación, es decir, es la interfaz de usuario que el usuario interactúa. Esta capa es responsable de mostrar los datos del modelo al usuario y permitir que el usuario interactúe con la aplicación.

- Controlador: El controlador es el intermediario entre la vista y el modelo. Es responsable de procesar las solicitudes del usuario, comunicarse con el modelo para obtener los datos necesarios y actualizar la vista en consecuencia. El controlador se encarga de la lógica de la aplicación y dirige el flujo de datos entre la vista y el modelo.

- Separación de responsabilidades: Una de las principales funcionalidades del modelo MVC es la separación de responsabilidades. Cada capa tiene una función claramente definida y está diseñada para ser independiente de las demás. Esto hace que la aplicación sea más fácil de mantener y modificar, ya que los cambios realizados en una capa no afectarán necesariamente a las demás capas.

- Flexibilidad: El modelo MVC proporciona una gran flexibilidad en el diseño y desarrollo de la aplicación. Cada capa puede ser modificada y mejorada de forma independiente sin afectar a las demás capas. Además, se pueden agregar nuevas funcionalidades y características a la aplicación sin necesidad de alterar el código existente.

- Reutilización de código: Otra funcionalidad importante del modelo MVC es la reutilización de código. La separación de responsabilidades y la independencia entre las capas permiten que el código sea reutilizado en diferentes partes de la aplicación. Esto reduce la cantidad de código necesario y hace que la aplicación sea más eficiente y fácil de mantener.

En resumen, el modelo MVC proporciona una forma eficiente y estructurada de desarrollar aplicaciones de software, con una clara separación de responsabilidades y una gran flexibilidad para el desarrollo y mantenimiento de la aplicación.


##Definir la infraestructura para el almacenamiento y recuperación de datos.
En el contexto del patrón de arquitectura MVC (Modelo-Vista-Controlador), la infraestructura de almacenamiento y recuperación de datos se refiere a la parte del modelo que se encarga de la gestión de los datos.

El modelo representa los datos y la lógica empresarial de una aplicación, y se encarga de almacenar, recuperar y manipular los datos según sea necesario. La infraestructura para el almacenamiento y recuperación de datos en el modelo de MVC incluye el diseño de la base de datos, la implementación de las operaciones CRUD (crear, leer, actualizar y eliminar) y la interacción con el controlador para proporcionar datos a la vista.

En este sentido, la infraestructura para el almacenamiento y recuperación de datos en el modelo de MVC puede incluir tecnologías de bases de datos, como MySQL, PostgreSQL, MongoDB, entre otras, así como el uso de ORMs (Object-Relational Mapping) para la manipulación de datos en la aplicación. También puede incluir la implementación de patrones de diseño como el patrón Repositorio para la gestión de datos en el modelo.

He aqui un ejemplo, utilizando la conexion a la base de datos: 

~~~
<?php 

define('DB_SERVER', 'localhost');
define('DB_NAME', 'aw2022');
define('DB_USER', 'root');
define('DB_PASS', '');

 ?>

<?php 
require 'config.php';

class BD_PDO
{
	//public $tot_reg;
	//public $ultimo_id;
	public function getConection ()	
	{
		try {
			    $conexion = new PDO("mysql:host=".DB_SERVER.";dbname=".DB_NAME.";",DB_USER,DB_PASS);
			       	// , 'array(PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"');
		}
		catch(PDOException $e)
		{
	         echo "Failed to get DB handle: " . $e->getMessage();
	         exit;    
	    }
	    return $conexion;
	}		

	public function Ejecutar_Instruccion($consulta_sql)
	{
		# code...
		$conexion = $this->getConection();
        $rows = array();        
		$query=$conexion->prepare($consulta_sql);
		if(!$query)
		{
         	return "Error al mostrar";
        }
		else
		{			
        	$query->execute();   
           	$this->tot_reg = $query->rowCount();     	
        	while ($result = $query->fetch())
			{
            	$rows[] = $result;
          	}			
            return $rows;
        }
	}


	public function listados($instruccion_sql,$instruccion_sql_foranea)
	{
		$option = "";
		$datos = $this->Ejecutar_Instruccion($instruccion_sql);

		$datos_foranea = $this->Ejecutar_Instruccion($instruccion_sql_foranea);
		foreach ($datos as $renglon) 
		{
			if (@$datos_foranea[0][0]==$renglon[0])
			{
				$selected = "Selected";
			}
			else
			{
				$selected = "";
			}
			$option=$option.'<option value="'.$renglon[0].'"'.$selected.'>'.$renglon[1].'</option>';
		}
		return $option;
	}	
}
?>
~~~



