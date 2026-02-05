i have this html   $('#newsForm').on('submit', function(e) {
            e.preventDefault(); // Prevent normal form submission

            // Create FormData object
            var formData = new FormData();

            // Get text inputs
            formData.append('title', $('input[name="title"]').val());



            // Get CKEditor data
            formData.append('description', CKEDITOR.instances.description.getData());

            // Get image file if exists
            const imageFile = $('#imageInput')[0].files[0];
            if (imageFile) {
                formData.append('image', imageFile);
            }

            // AJAX call to PHP
            $.ajax({
                url: 'assets/php/news/add_news.php', // your PHP script
                type: 'POST',
                data: formData,
                processData: false, // Important for FormData
                contentType: false, // Important for FormData
                success: function(response) {
                    console.log(response); // For debugging
                    Swal.fire({
                        icon: 'success',
                        title: 'Actualité enregistrée !',
                        text: 'La news a été ajoutée avec succès.'
                    });

                    // Optional: reset form
                    $('#newsForm')[0].reset();
                    $('#imagePreview').hide();
                    CKEDITOR.instances.description.setData('');
                },
                error: function(xhr, status, error) {
                    console.error(xhr.responseText);
                    Swal.fire({
                        icon: 'error',
                        title: 'Erreur',
                        text: 'Une erreur est survenue lors de l\'enregistrement.'
                    });
                }
            });
        }); and this table news : 1	id Primaire	int(11)			Non	Aucun(e)		AUTO_INCREMENT	Modifier Modifier	Supprimer Supprimer	
	2	title	varchar(255)	utf8mb4_general_ci		Non	Aucun(e)			Modifier Modifier	Supprimer Supprimer	
	3	date	date			Non	Aucun(e)			Modifier Modifier	Supprimer Supprimer	
	4	description	text	utf8mb4_general_ci		Non	Aucun(e)			Modifier Modifier	Supprimer Supprimer	
	5	image	varchar(255)	utf8mb4_general_ci		Oui	NULL			Modifier Modifier	Supprimer Supprimer	
	6	link	varchar(255)	utf8mb4_general_ci		Oui	NULL			Modifier Modifier	Supprimer Supprimer	  i need php code , this is structure of php codes : <?php
session_start();
require_once '../config.php'; // adapte si besoin
try {
    $bdd = new PDO(
        "mysql:host=".DB_SERVER.";dbname=".DB_NAME.";charset=utf8mb4",
        DB_USER,
        DB_PASS,
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );

// Sécurité : admin uniquement
if (!isset($_SESSION['role']) || $_SESSION['role'] !== 'admin') {
    http_response_code(403);
    echo json_encode([]);
    exit;
}

$sql = "SELECT 
            id,
            title,
            description,
            image,
            link,
            DATE_FORMAT(date, '%d/%m/%Y') AS date_fr
        FROM news
        ORDER BY date DESC";

$stmt = $bdd->prepare($sql);
$stmt->execute();

$news = $stmt->fetchAll(PDO::FETCH_ASSOC);

echo json_encode($news);

} catch (Exception $e) {

    if ($bdd->inTransaction()) {
        $bdd->rollBack();
    }

    echo json_encode([
        'response' => 'false',
        'message'  => 'exception',
        'error'    => $e->getMessage()
    ]);
}
