# Proposition de code pour une architecture MVC !

Classe de base implémentant le singleton pour la connexion à la base de données

# Fichier classes/bdd_mysql.class.php

```php runnable
<?php

class bdd_mysql{
	
	private static $_laconnexion;
	
	/*
	*	Constructeur de BDD
	*/
	function __construct($_nomdb, $_user, $_pass){
		
		$con=new PDO("mysql:host=127.0.0.1;dbname=$_nomdb", $_user, $_pass, array(PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING));
		bdd_mysql::$_laconnexion=$con;
	}
	
	public function getConnexion(){
		return bdd_mysql::$_laconnexion;
	}
}

?>
```

# Fichier index.php

```php runnable
<?php
	require_once("classes\bdd_mysql.class.php");
	
	/*
	*CREATE --> Insert
	*READ --> Select   --> Cas par défaut
	*UPDATE
	*DELETE
	*/
	
	
	$madb=new bdd_mysql("debutterphp","root","");
	
	
	var_dump($_GET);
	
	if(isset($_GET["action"])){
		$action=$_GET["action"];
		switch($action){
		
		case "create":
			//echo "affiche le formulaire";
			include('vues/form.php');
			break;
		case "delete":
			include('controller/delete.php');
			break;
		default:
			/*
			*	Gestion du listing
			*/
			include('vues/listing.php');
		}
	}
	else{
		/*
		*	Gestion du listing
		*/
		include('vues/accueil.php');
	}

?>

```
# vues/accueil.php
```php runnable
<h1>Welcome</h1>

<ul>
	<li>
		<a href="index.php?action=listing">Lister la base</a>
	</li>
	<li>
		<a href="index.php?action=create">Ajouter un prof</a>
	</li>
</ul>

```

# Fichier vues/listing.php
```php runnable
<?php

	//Toujours debugguer sa requete SQL en la plaçant dans une variable
	$sql="SELECT * from profs WHERE supprimer=0";
	//var_dump($sql);
	
	//Exécution de la requete et recupération du jeu d'enregistrement (aussi appelé Curseur)
	$jeu=$madb->getConnexion()->query($sql);
?>	

	<script language="javascript">
		function confirmation(unid, unnom, unprenom){
			console.log(unid);
			if ( confirm( "voulez vous supprimez cet enregistrement " + unnom + " " + unprenom ) ) {
				document.location.href="index.php?action=delete&id=" + unid;
			} 
		}
	</script>
	
	
	<h1>Listing</h1>
	<table border="1">
		<tr>
			<th></th>
			<th>id</th>
			<th>nom</th>
			<th>prenom</th>
		</tr>
		<?php
		//Itération Pour chaque Ligne (Row) du Jeu d'enregistrement($jeu)
		foreach($jeu as $row){?>
		<tr>
			<td><a href="#" onclick="confirmation(<?php echo $row["id"]?>,'<?php echo $row["nom"]?>','<?php echo $row["prenom"]?>')"> supprimer</a></td>
			<td><?php echo $row["id"] ?></td>
			<td><?php echo $row["nom"]  ?></td>
			<td><?php echo $row["prenom"]  ?></td>
		</tr>
		<?php } ?>
	</table>
	
	<h2><a href="index.php?action=create">Insérer un nouvel enregisrement</a>
```
# Fichier vues/form.php
```php runnable
<?php
	if(isset($_POST["bt_submit"])){
		include("controller/create.php");
	}
	else{
		//Affiche le formulaire
?>
	<form action="index.php?action=create" method="POST">
		
		<label for="_id">Identifiant</label>
		<input id="_id" type="text" name="txt_id">
		
		<br/>
		
		<label for="_nom">Nom</label>
		<input id="_nom" type="text" name="txt_nom">
		
		<br/>
		
		<label for="_prenom">Prenom</label>
		<input id="_prenom" type="text" name="txt_prenom">
		
		<br/>
			
		<input type="submit" name="bt_submit" value="Valider">
		
	</form>
	<?php
	}
	?>
```
# Fichier controller/form.php
```php runnable
<?php
	var_dump($_POST);
	
	// Cas du Insert
	$_id=htmlentities($_POST["txt_id"]);
	$_nom=htmlentities($_POST["txt_nom"]);
	$_prenom=htmlentities($_POST["txt_prenom"]);
	
	
	$sql="INSERT INTO profs VALUES(:id,:nom,:prenom, 0)";
	//var_dump($sql);
	
	$stmt=$madb->getConnexion()->Prepare($sql);
	
	$stmt->bindParam(':id', $_id);
	$stmt->bindParam(':nom', $_nom);
	$stmt->bindParam(':prenom', $_prenom);
	
	$stmt->execute();
	
	header("Location: index.php?action=listing");  
?>
```
# Fichier controller/delete.php
```php runnable
<?php
	//var_dump($_GET);
	
	// Cas du Insert
	$_id=$_GET["id"];

    //On simule un delete en utilisant un booleen
	$sql="UPDATE profs SET supprimer=1 WHERE id=$_id";
	//var_dump($sql);
	
	$stmt=$madb->getConnexion()->Exec($sql);
	
	
	header("Location: index.php?action=listing");  
?>
```