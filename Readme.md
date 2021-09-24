OCamlFormat
OCamlFormat es una herramienta para formatear código OCaml.

OCamlFormat funciona analizando el código fuente utilizando el analizador estándar del compilador OCaml, decidiendo dónde colocar los comentarios en el árbol de análisis y escribiendo el árbol de análisis y los comentarios en un estilo coherente.

Consulte el código fuente de OCamlFormat e Infer para obtener ejemplos de los estilos de código que produce.

Tabla de contenido
Preguntas frecuentes para nuevos usuarios
Características
Visión general
Estilo de código
Opciones
Instalación
Configuración del editor
Configuración de Emacs
Configuración de Vim
Documentación
Comunidad
Licencia
Preguntas frecuentes para nuevos usuarios
¡Hola, nuevo usuario! ¡Bienvenido!👋

Si está aquí, probablemente esté interesado en utilizar una herramienta de formato para su base de código, de modo que no tenga que preocuparse por formatearlo a mano y para acelerar la revisión del código centrándose en las partes importantes.

¡OCamlFormat es probablemente lo que buscas! Pero hay algunas cosas que debe saber antes de formatear todas las cosas.

¿Debería usar OCamlFormat?
OCamlFormat ya está siendo utilizado por varios proyectos, pero viene con algunas advertencias importantes. Estas preguntas frecuentes deberían ayudarlo a decidir si puede funcionar para usted.

OCamlFormat es un software beta. Si bien no seguimos SemVer , esperamos que el programa cambie considerablemente antes de llegar a la versión 1.0.0. En particular, actualizar el ocamlformatpaquete hará que su programa se vuelva a formatear. A veces es relativamente indoloro, pero a veces marcará una diferencia en casi todos los archivos. Este puede ser un precio difícil de pagar, ya que significa perder el historial de git correspondiente.

Si usa una configuración personalizada, las opciones en las que confía también pueden eliminarse en una versión posterior.

Además, si adopta OCamlFormat en un proyecto, no interrumpirá su flujo de trabajo en sus otros proyectos. De hecho, OCamlFormat modifica un archivo solo si puede encontrar un .ocamlformatarchivo, por lo que agregar un gancho de guardado en su editor solo simplificará su flujo de trabajo en proyectos que usan OCamlFormat.

¿Qué configuración debo utilizar?
La forma recomendada es utilizar un perfil predeterminado versionado, como:

version=0.19.0
(o reemplazar con la salida de ocamlformat --version)

Esto asegura dos cosas:

está utilizando la configuración de formato predeterminada.
la versión que usas para formatear está registrada en algún lugar. Si alguien más que trabaja en el proyecto intenta usar una versión diferente, verá un mensaje de error en lugar de reformatear todo el proyecto de una manera diferente.
¿Puede OCamlFormat apoyar mi estilo?
No. Es mejor ver OCamlFormat como una herramienta para aplicar un estilo, en lugar de una herramienta modificable para hacer cumplir su estilo existente. Hay algunas perillas que puede girar, como anular marginpara determinar el ancho máximo de línea. Pero es mejor no establecer opciones individuales para anular lo que está haciendo el perfil predeterminado.

Para citar (y sed) la página de más bonita sobre la filosofía de las opciones :

OCamlFormat tiene algunas opciones debido al historial. Pero no queremos más de ellos.

Con mucho, la razón más importante para adoptar OCamlFormat es detener todos los debates en curso sobre estilos.

Cuantas más opciones tenga OCamlFormat, más se alejará del objetivo anterior. Los debates sobre estilos simplemente se convierten en debates sobre qué opciones de OCamlFormat usar.

¿Cómo deshabilitar localmente OCamlFormat?
Para deshabilitar el formato de un elemento de nivel superior específico, debe adjuntar un [@@ocamlformat "option=VAL"]atributo a este elemento en el archivo procesado, como por ejemplo:

dejar  do_not_touch 
    ( x  : t )
      ( y  : t )
        ( z  : t ) = [
  X; y; z
] [ @@ ocamlformat " deshabilitar " ]
Para deshabilitar el formato de una expresión específica, debe adjuntar un [@ocamlformat "option=VAL"]atributo a esta expresión en el archivo procesado, como por ejemplo:

let  do_not_touch ( x  : t ) ( y  : t ) ( z  : t ) = [
  X; y; z
] [ @ ocamlformat " deshabilitar " ]
Para deshabilitar un archivo completo, la forma preferida es agregar el nombre del archivo a un .ocamlformat-ignorearchivo local . Un .ocamlformat-ignorearchivo especifica archivos que OCamlFormat debe ignorar. Cada línea de un .ocamlformat-ignorearchivo especifica un nombre de archivo relativo al directorio que contiene el .ocamlformat-ignorearchivo. Se admiten expresiones regulares de estilo shell. Las líneas que comienzan con #se ignoran y se pueden utilizar como comentarios.

Características
Visión general
OCamlFormat requiere un código fuente que cumpla las siguientes condiciones:

No activa la advertencia 50 ("Comentario de documentación inesperado"). Para el código que activa la advertencia 50, es poco probable que OCamlFormat conserve la cadena de documentación adjunta.

Analiza sin ningún preprocesamiento, utilizando la versión del analizador OCaml estándar (no camlp4) utilizado para compilar OCamlFormat. Los atributos y puntos de extensión deben conservarse correctamente, pero otros mecanismos como camlp4, cppo, etc. no funcionarán.

Es una implementación de módulo ( .ml), una interfaz ( .mli) o una secuencia de frases de nivel superior ( .mlt). Los archivos de dunas en sintaxis OCaml también funcionan.

En esas condiciones, se espera que OCamlFormat produzca una salida equivalente a la entrada. Como comprobación de seguridad en caso de errores, antes de finalizar o modificar cualquier archivo de entrada, OCamlFormat aplica las siguientes comprobaciones:

Los árboles de análisis obtenidos al analizar los archivos originales y formateados son iguales hasta una pequeña normalización (ver ).Normalize.equal

Se han conservado las cadenas de documentación y su adjunto (implícito en la verificación del árbol de análisis).

El conjunto de comentarios en los archivos originales y formateados es el mismo hasta su ubicación.

Estilo de código
Hay una serie de perfiles de estilo de código preestablecidos, que se seleccionan mediante la --profileopción pasando --profile=<name>la línea de comandos o agregándolos profile = <name>a un archivo de configuración .ocamlformat. Cada perfil es una colección de configuraciones para todas las opciones, anulando la configuración de menor prioridad de las opciones individuales. Por lo tanto, se puede seleccionar un perfil y luego se pueden anular las opciones individuales si se desea.

El perfil conventionalo defaultpretende ser tan familiar y "convencional" que aparezcan como lo permitan las opciones disponibles.

El ocamlformatperfil tiene como objetivo aprovechar las fortalezas de un formateador automático basado en árbol de análisis y limitar las consecuencias de las debilidades impuestas por la implementación actual. Este es un estilo que se optimiza para lo que el formateador puede hacer mejor, en lugar de coincidir con el estilo de cualquier código existente. En lugar de familiaridad, la atención se centra en la legibilidad, manteniendo los casos comunes razonablemente compactos mientras se intenta evitar el formato confuso en casos de esquina. Las pautas generales que han dirigido el diseño incluyen:

La legibilidad, en el sentido de dificultar al máximo el análisis visual rápido para dar una interpretación incorrecta, es de máxima prioridad;

Siempre que sea posible, la estructura de alto nivel del código debería ser obvia mirando solo el margen izquierdo, en particular, no debería ser necesario saltar visualmente de izquierda a derecha buscando palabras clave críticas, tokens, etc.

Se prefiere todo lo demás código compacto igual, ya que la lectura sin desplazamiento es más fácil, por lo que se evita la sangría o los espacios en blanco a menos que ayude a la legibilidad;

Se ha prestado atención a hacer que algunas trampas sintácticas sean visualmente obvias.

El compactperfil es similar ocamlformatpero opta por un estilo de código generalmente más compacto.

El sparseperfil es similar ocamlformatpero opta por un estilo de código generalmente más escaso.

Si no se selecciona ningún perfil, conventionalse utiliza el uno.

Opciones
La documentación de las opciones completas está disponible en [ocamlformat-help.txt] y hasta ocamlformat --help. Las opciones se pueden modificar mediante:

un archivo de configuración .ocamlformat con una option = VALlínea
usando la OCAMLFORMATvariable de entorno:OCAMLFORMAT=option=VAL,...,option=VAL
un parámetro opcional en la línea de comando
un [@@@ocamlformat "option=VAL"]atributo global en el archivo procesado
un [@@ocamlformat "option=VAL"]atributo en una expresión en el archivo procesado
Se utilizan los archivos .ocamlformat en los directorios que contienen y todos los ancestros para cada archivo de entrada, así como el archivo .ocamlformat global definido en $XDG_CONFIG_HOME/ocamlformat. El archivo .ocamlformat global tiene la prioridad más baja, entonces cuanto más cerca esté el directorio del archivo procesado, mayor será la prioridad.

Cuando la opción --enable-outside-detected-projectno está configurada, los archivos .ocamlformat fuera del proyecto (incluido el que está dentro XDG_CONFIG_HOME) no se leen. La raíz del proyecto de un archivo de entrada se toma como el directorio ancestro más cercano que contiene un archivo .git o .hg o dune-project. Si no se encuentra ningún archivo de configuración, el formateo está deshabilitado.

Un .ocamlformat-ignorearchivo especifica archivos que OCamlFormat debe ignorar. Cada línea de un .ocamlformat-ignorearchivo especifica un nombre de archivo relativo al directorio que contiene el .ocamlformat-ignorearchivo. Las líneas que comienzan con #se ignoran y se pueden utilizar como comentarios.

Instalación
OCamlFormat se puede instalar con opam:

opam install ocamlformat
Alternativamente, consulte las ocamlformat.opaminstrucciones de construcción manual.

Configuración del editor
Desactivar proyecto externo
Como se menciona en la sección Opciones, cuando --disable-outside-detected-projectse establece la opción , los archivos .ocamlformat fuera del proyecto (incluido el que está dentro XDG_CONFIG_HOME) no se leen. La raíz del proyecto de un archivo de entrada se toma como el directorio ancestro más cercano que contiene un archivo .git o .hg o dune-project. Si no se encuentra ningún archivo de configuración, el formateo está deshabilitado.

Esta función suele ser el comportamiento que puede esperar de OCamlFormat cuando se ejecuta directamente desde su editor de texto, por lo que se recomienda utilizar esta opción.

Configuración de Emacs
agregar $(opam config var share)/emacs/site-lispa load-path(hecho por opam user-setup install)

agregar (require 'ocamlformat)a.emacs

opcionalmente, agregue lo siguiente para .emacsenlazar C-M-<tab>con el comando ocamlformat e instale un gancho para ejecutar ocamlformat al guardar:

( agregar gancho  'tuareg-mode-hook ( lambda ()
  ( define-key tuareg-mode-map ( kbd  " CM- <tab> " ) # 'ocamlformat )
  ( agregar-gancho  'antes-guardar-gancho  # ' ocamlformat-antes-guardar )))
Para pasar la opción --disable-outside-detected-project(o --disable) a OCamlFormat:

correr emacs
corre y M-x customize-group⏎luego entraocamlformat⏎
seleccione el elemento Habilitar Ocamlformat
seleccione el modo OCamlformat en el menú Valor: Enable(por defecto), DisableoDisable outside detected project
guardar el búfer ( C-x C-s) luego entrar yes⏎y salir
Se pueden configurar otras opciones de OCamlFormat en archivos de configuración .ocamlformat.

Con paquete de uso
Una configuración básica con use-package :

( use-package ocamlformat
   : custom (ocamlformat-enable 'enable-outside-detectado-project )
   : hook (antes de guardar . ocamlformat-before-save)
  )
A veces es necesario tener un conmutador para OCamlFormat (debido a conflictos de versiones o porque no desea instalarlo en todos los conmutadores, por ejemplo). Teniendo en cuenta que su conmutador OCamlFormat se llama ocamlformat:

( use-package ocamlformat
   : load-path 
  ( lambda ()
    ( Concat 
         ; , el uso Nunca "/" o "\" ya que esto no es portátil (OPAM-usuario-configuración hace esto sin embargo) 
         ; ; Siempre uso de nombre de archivo-como-directorio ya que esto añadirá el separador correcto, si es necesario 
         ; ; (o utilizar un paquete que lo hace bien como https://github.com/rejeep/f.el) 
         ; ; Este es el detallado y no paquete de la versión en función: 
         ( nombre-archivo-como-directorio 
          ; ; no se pudo encontrar una opción para eliminar la nueva línea, por lo que se necesita una subcadena 
          ( subcadena ( shell-command-to-string  " opam config var share --switch = ocamlformat --safe " ) 0  -1 ))
         ( nombre-archivo-como-directorio  " emacs " )
         ( nombre-archivo-como-directorio  " site-lisp " )))
   : personalizado 
  (ocamlformat-enable 'enable-outside-detectado-project )
  (comando ocamlformat
   ( concat 
    ( file-name-as-directory 
     ( substring ( shell-command-to-string  " opam config var bin --switch = ocamlformat --safe " ) 0  -1 ))
     " ocamlformat " ))
   : hook (before- guardar . ocamlformat-before-save)
  )
(Observe el :custompara personalizar el binario OCamlFormat)

Esto podría simplificarse (definiendo una variable elisp correspondiente al prefijo del conmutador al cargar tuareg, por ejemplo) pero permite tener una configuración completa en un solo lugar, lo que a menudo es menos propenso a errores.

Configuración de Vim
asegúrese de que el ocamlformatbinario se pueda encontrar en PATH

instalar el complemento Neoformat

Opcional: puede cambiar las opciones pasadas a OCamlFormat (para usar la opción, --disable-outside-detected-projectpor ejemplo), puede personalizar NeoFormat con:

let g:opambin = substitute(system('opam config var bin'),'\n$','','''')
let g:neoformat_ocaml_ocamlformat = {
            \ 'exe': g:opambin . '/ocamlformat',
            \ 'no_append': 1,
            \ 'stdin': 1,
            \ 'args': ['--disable-outside-detected-project', '--name', '"%:p"', '-']
            \ }

let g:neoformat_enabled_ocaml = ['ocamlformat']
Documentación
OCamlFormat está documentado en su página de manual y a través de su ayuda interna:

ocamlformat --help
man ocamlformat
También puede verlo en línea .

Razón
OCamlFormat está influenciado y sigue el mismo diseño básico que refmtpara Reason , pero genera OCaml en lugar de Reason.

Esta herramienta no puede tratar directamente con el código Reason ( *.re/ *.reiarchivos), pero es posible convertir primero estos archivos a la sintaxis OCaml usando refmt -p mly luego ejecutándose ocamlformaten esta salida.

Comunidad
foro: https://discuss.ocaml.org/tags/ocamlformat
github: https://github.com/ocaml-ppx/ocamlformat
problemas: https://github.com/ocaml-ppx/ocamlformat/issues
Vea CONTRIBUYENDO para saber cómo ayudar.

Licencia
OCamlFormat tiene licencia del MIT .
