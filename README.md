redactor
========

Sistema Wysiwyg para YII usando Redactor (Extensión)

Reemplazaremos el campo en _form con este código donde queremos que aparezca el código:

 //elemento
 <?php echo $form->labelEx($model,'contenido'); ?>
 <?php $this->widget(
     'ext.redactor.ERedactorWidget', array(
           'options' => array(
           'lang'=>'es',
           'imageUpload' => Yii::app()->createAbsoluteUrl('contenidos/ImageUpload'),
           'imageGetJson' => Yii::app()->createAbsoluteUrl('contenidos/ListImages')
        ),
        'model' => $model,
        'attribute' => 'contenido',
     ));
 ?>
 <?php echo $form->error($model,'contenido'); ?>
 
Debemos cerciorarnos que en donde definimos la ruta Absoluta, esté nuestro controlador.

Luego agregamos esto en nuestro controlador:

	// para subir imágenes
	public function actionImageUpload() {
        $uploadedFile = CUploadedFile::getInstanceByName('file');
        if (!empty($uploadedFile)) {
            //$rnd = rand();  // genera número aleatorio entre 0-9999
            $rnd=date("Ymdhis"); // genera el nombre basado en añomesdiahora
            $fileName = "{$rnd}.{$uploadedFile->extensionName}";  // random number + file name
            if ($uploadedFile->saveAs(Yii::app()->basePath.'/../uploads/' . $fileName)) {
                echo CHtml::image(Yii::app()->request->getBaseUrl(true).'/uploads/' . $fileName);
                Yii::app()->end();
            }
        }
        throw new CHttpException(400, 'Algo salió mal!');
    }

	// "ListImages" (usado para ver las imágenes ya subidas)
    public function actionListImages() {
        $images = array();
        $handler = opendir(Yii::app()->basePath.'/../uploads/');
        while ($file = readdir($handler)) {
            if ($file != "." && $file != "..")
                $images[] = $file;
        }
        closedir($handler);
        $jsonArray = array();
        foreach ($images as $image)
            $jsonArray[] = array(
                'thumb' => Yii::app()->request->getBaseUrl(true).'/uploads/' . $image,
                'image' => Yii::app()->request->getBaseUrl(true) . '/uploads/' . $image,
            );
        header('Content-type: application/json');
        echo CJSON::encode($jsonArray);
    }

Debemos tener muy en cuenta que en la base del app generada con Yii debemos tener una carpeta llamada uploads (o cualquiera que crees), solo es importante indicarla.

Con esto debe funcionar bien.