Formato OCaml
OCamlFormat es una herramienta para formatear el código OCaml.

OCamlFormat funciona analizando el código fuente utilizando el analizador estándar del compilador OCaml, decidiendo dónde colocar los comentarios en el árbol de análisis y escribiendo el árbol de análisis y los comentarios en un estilo coherente.

Consulte el código fuente de OCamlFormat e Infer para ver ejemplos de los estilos de código que producen.

tabla de contenido
Preguntas frecuentes para nuevos usuarios
caracteristicas
Descripcion general
Estilo de código
Opciones
instalacion
Configuración del editor
Configuración de Emacs
configuración vim
documentacion
Comunidad
Licenciado
Preguntas frecuentes para nuevos usuarios
¡Hola, nuevo usuario! ¡Bienvenido!👋

Si está aquí, probablemente esté interesado en usar una herramienta de formato para su código base, de modo que no tenga que preocuparse por formatearlo a mano y acelerar la revisión del código centrándose en las partes importantes.

¡OCamlFormat es probablemente lo que buscas! Pero hay algunas cosas que debe saber antes de formatear todas las cosas.

¿Debo usar OCamlFormat?
OCamlFormat ya está siendo utilizado por varios proyectos, pero viene con algunas advertencias importantes. Estas preguntas frecuentes deberían ayudarlo a decidir si puede funcionar para usted.

OCamlFormat es un software beta. Si bien no seguimos a SemVer , esperamos que el programa cambie considerablemente antes de llegar a la versión 1.0.0. En particular, actualizar el ocamlformatpaquete hará que su programa se vuelva a formatear. A veces es relativamente indoloro, pero a veces hará una diferencia en casi todos los archivos. Este puede ser un precio difícil de pagar, ya que significa perder el historial de git correspondiente.

Si usa una configuración personalizada, es posible que las opciones en las que confía también se eliminen en una versión posterior.

Además, si adopta OCamlFormat en un proyecto, no interrumpirá su flujo de trabajo en sus otros proyectos. De hecho, OCamlFormat modifica un archivo solo si puede encontrar un .ocamlformatarchivo, por lo que agregar un gancho para guardar en su editor solo simplificará su flujo de trabajo en proyectos que usan OCamlFormat.

¿Qué configuración debo usar?
La forma recomendada es utilizar un perfil predeterminado versionado, como:

version=0.19.0
(o reemplazar con la salida de ocamlformat --version)

Esto asegura dos cosas:

está utilizando la configuración de formato predeterminada.
la versión que usa para formatear está grabada en alguna parte. Si alguien más que trabaja en el proyecto intenta usar una versión diferente, verá un mensaje de error en lugar de reformatear todo el proyecto de una manera diferente.
¿Puede OCamlFormat admitir mi estilo?
No. Es mejor ver OCamlFormat como una herramienta para aplicar un estilo, en lugar de una herramienta modificable para aplicar su estilo existente. Hay algunas perillas que puede girar, como anular marginpara determinar el ancho de línea máximo. Pero es mejor no establecer opciones individuales para anular lo que está haciendo el perfil predeterminado.

Para citar (y sed) la página de Prettier sobre la filosofía de las opciones :

OCamlFormat tiene algunas opciones debido a la historia. Pero no queremos más de ellos.

Con mucho, la principal razón para adoptar OCamlFormat es detener todos los debates en curso sobre los estilos.

Cuantas más opciones tiene OCamlFormat, más se aleja del objetivo anterior. Los debates sobre estilos simplemente se convierten en debates sobre qué opciones de OCamlFormat usar.

¿Cómo deshabilitar localmente OCamlFormat?
Para deshabilitar el formato de un elemento de nivel superior específico, debe adjuntar un [@@ocamlformat "option=VAL"]atributo a este elemento en el archivo procesado, como:

let do_not_touch
    (x : t)
      (y : t)
        (z : t) = [
  x; y; z
] [@@ocamlformat "disable"]
Para deshabilitar el formato de una expresión específica, debe adjuntar un [@ocamlformat "option=VAL"]atributo a esta expresión en el archivo procesado, como:

let do_not_touch (x : t) (y : t) (z : t) = [
  x; y; z
] [@ocamlformat "disable"]
Para deshabilitar un archivo completo, la forma preferida es agregar el nombre del archivo a un .ocamlformat-ignorearchivo local. Un .ocamlformat-ignorearchivo especifica archivos que OCamlFormat debe ignorar. Cada línea de un .ocamlformat-ignorearchivo especifica un nombre de archivo relativo al directorio que contiene el .ocamlformat-ignorearchivo. Se admiten expresiones regulares de estilo Shell. Las líneas que comienzan con #se ignoran y se pueden usar como comentarios.

Características
Descripción general
OCamlFormat requiere un código fuente que cumpla con las siguientes condiciones:

No activa la advertencia 50 ("Comentario de documentación inesperado"). Para el código que activa la advertencia 50, es poco probable que OCamlFormat conserve la cadena de documentación adjunta.

Analiza sin ningún procesamiento previo, utilizando la versión del analizador estándar OCaml (no camlp4) que se usa para compilar OCamlFormat. Los atributos y los puntos de extensión deben conservarse correctamente, pero otros mecanismos como camlp4, cppo, etc. no funcionarán.

Es una implementación de módulo ( .ml), una interfaz ( .mli) o una secuencia de frases de alto nivel ( .mlt). Los archivos de dunas en la sintaxis de OCaml también funcionan.

Bajo esas condiciones, se espera que OCamlFormat produzca una salida equivalente a la entrada. Como control de seguridad en caso de errores, antes de finalizar o modificar cualquier archivo de entrada, OCamlFormat aplica los siguientes controles:

Los árboles de análisis obtenidos al analizar los archivos originales y formateados son iguales, salvo por alguna normalización menor (ver ).Normalize.equal

Las cadenas de documentación y sus archivos adjuntos se han conservado (implícito en la comprobación del árbol de análisis).

El conjunto de comentarios en los archivos originales y formateados es el mismo hasta su ubicación.

Estilo de código
Hay una serie de perfiles de estilo de código preestablecidos, seleccionados mediante la --profileopción pasando --profile=<name>la línea de comando o agregando profile = <name>a un archivo de configuración .ocamlformat. Cada perfil es una colección de configuraciones para todas las opciones, anulando la configuración de menor prioridad de las opciones individuales. Por lo tanto, se puede seleccionar un perfil y luego se pueden anular las opciones individuales si se desea.

El perfil conventionalo defaultpretende ser tan familiar y "convencional" como lo permitan las opciones disponibles.

El ocamlformatperfil tiene como objetivo aprovechar las ventajas de un formateador automático basado en árboles de traducción y limitar las consecuencias de las debilidades impuestas por la implementación actual. Este es un estilo que optimiza lo que el formateador puede hacer mejor, en lugar de coincidir con el estilo de cualquier código existente. En lugar de familiaridad, la atención se centra en la legibilidad, manteniendo los casos comunes razonablemente compactos mientras se intenta evitar el formato confuso en los casos de esquina. Las pautas generales que han dirigido el diseño incluyen:

La legibilidad, en el sentido de hacer lo más difícil posible que un análisis visual rápido dé una interpretación incorrecta, es de máxima prioridad;

Siempre que sea posible, la estructura de alto nivel del código debe ser obvia mirando solo el margen izquierdo, en particular, no debe ser necesario saltar visualmente de izquierda a derecha buscando palabras clave críticas, tokens, etc.

Se prefiere el código compacto igual a todo lo demás, ya que la lectura sin desplazamiento es más fácil, por lo que se evitan las sangrías o los espacios en blanco a menos que ayuden a la legibilidad;

Se ha prestado atención a hacer que algunas trampas sintácticas sean visualmente obvias.

El compactperfil es similar ocamlformatpero opta por un estilo de código generalmente más compacto.

El sparseperfil es similar ocamlformatpero opta por un estilo de código generalmente más escaso.

Si no se selecciona ningún perfil, conventionalse utiliza uno.

Opciones
La documentación completa de las opciones está disponible en [ocamlformat-help.txt] ya través de ocamlformat --help. Las opciones se pueden modificar mediante:

un archivo de configuración .ocamlformat con una option = VALlínea
usando la OCAMLFORMATvariable de entorno:OCAMLFORMAT=option=VAL,...,option=VAL
un parámetro opcional en la línea de comando
un atributo global [@@@ocamlformat "option=VAL"]en el archivo procesado
un [@@ocamlformat "option=VAL"]atributo en una expresión en el archivo procesado
Se utilizan archivos .ocamlformat en los directorios contenedor y todos los ancestros para cada archivo de entrada, así como el archivo global .ocamlformat definido en $XDG_CONFIG_HOME/ocamlformat. El archivo global .ocamlformat tiene la prioridad más baja, entonces cuanto más cerca esté el directorio del archivo procesado, mayor será la prioridad.

Cuando la opción --enable-outside-detected-projectno está configurada, los archivos .ocamlformat fuera del proyecto (incluido el que está en XDG_CONFIG_HOME) no se leen. La raíz del proyecto de un archivo de entrada se toma como el directorio principal más cercano que contiene un archivo .git o .hg o dune-project. Si no se encuentra ningún archivo de configuración, el formateo está deshabilitado.

Un .ocamlformat-ignorearchivo especifica archivos que OCamlFormat debe ignorar. Cada línea de un .ocamlformat-ignorearchivo especifica un nombre de archivo relativo al directorio que contiene el .ocamlformat-ignorearchivo. Las líneas que comienzan con #se ignoran y se pueden usar como comentarios.

Instalación
OCamlFormat se puede instalar con opam:

opam install ocamlformat
Alternativamente, consulte ocamlformat.opamlas instrucciones de compilación manual.

Configuración del editor
Deshabilitar proyecto externo
Como se mencionó en la sección Opciones, cuando la opción --disable-outside-detected-projectestá configurada, los archivos .ocamlformat fuera del proyecto (incluido el que está en XDG_CONFIG_HOME) no se leen. La raíz del proyecto de un archivo de entrada se toma como el directorio principal más cercano que contiene un archivo .git o .hg o dune-project. Si no se encuentra ningún archivo de configuración, el formateo está deshabilitado.

Esta función suele ser el comportamiento que puede esperar de OCamlFormat cuando se ejecuta directamente desde su editor de texto, por lo que se recomienda utilizar esta opción.

Configuración de Emacs
añadir $(opam config var share)/emacs/site-lispa load-path(como hecho por opam user-setup install)

añadir (require 'ocamlformat)a.emacs

Opcionalmente, agregue lo siguiente para .emacsenlazar C-M-<tab>con el comando ocamlformat e instale un enlace para ejecutar ocamlformat al guardar:

(add-hook 'tuareg-mode-hook (lambda ()
  (define-key tuareg-mode-map (kbd "C-M-<tab>") #'ocamlformat)
  (add-hook 'before-save-hook #'ocamlformat-before-save)))
Para pasar la opción --disable-outside-detected-project(o --disable) a OCamlFormat:

correremacs
corre M-x customize-group⏎luego entraocamlformat⏎
seleccione el elemento Habilitar formato Ocaml
seleccione el modo OCamlformat en el menú de valores: Enable(por defecto), DisableoDisable outside detected project
guardar el búfer ( C-x C-s), luego entrar yes⏎y salir
Se pueden configurar otras opciones de OCamlFormat en los archivos de configuración .ocamlformat.

Con paquete de uso
Una configuración básica con use-package :

(use-package ocamlformat
  :custom (ocamlformat-enable 'enable-outside-detected-project)
  :hook (before-save . ocamlformat-before-save)
  )
A veces, necesita tener un modificador para OCamlFormat (debido a conflictos de versión o porque no desea instalarlo en todos los modificadores, por ejemplo). Teniendo en cuenta que su interruptor OCamlFormat se llama ocamlformat:

(use-package ocamlformat
  :load-path
  (lambda ()
    (concat
         ;; Never use "/" or "\" since this is not portable (opam-user-setup does this though)
         ;; Always use file-name-as-directory since this will append the correct separator if needed
         ;; (or use a package that does it well like https://github.com/rejeep/f.el)
         ;; This is the verbose and not package depending version:
         (file-name-as-directory
          ;; Couldn't find an option to remove the newline so a substring is needed
          (substring (shell-command-to-string "opam config var share --switch=ocamlformat --safe") 0 -1))
         (file-name-as-directory "emacs")
         (file-name-as-directory "site-lisp")))
  :custom
  (ocamlformat-enable 'enable-outside-detected-project)
  (ocamlformat-command
   (concat
    (file-name-as-directory
     (substring (shell-command-to-string "opam config var bin --switch=ocamlformat --safe") 0 -1))
    "ocamlformat"))
  :hook (before-save . ocamlformat-before-save)
  )
(Observe el :custompara personalizar el binario OCamlFormat)

Esto podría simplificarse (definiendo una variable elisp correspondiente al prefijo switch al cargar tuareg, por ejemplo), pero permite tener una configuración completa en un solo lugar, lo que a menudo es menos propenso a errores.

configuración vim
asegúrese de que el ocamlformatbinario se pueda encontrar en PATH

instalar el plugin de Neoformato

Opcional: Puedes cambiar las opciones pasadas a OCamlFormat (para usar la opción --disable-outside-detected-projectpor ejemplo), puedes personalizar NeoFormat con:

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
OCamlFormat está influenciado y sigue el mismo diseño básico que refmtReason , pero genera OCaml en lugar de Reason.

Esta herramienta no puede tratar directamente con el código Reason ( archivos *.re/ *.rei), pero es posible convertir primero estos archivos a la sintaxis OCaml utilizando refmt -p mly luego ejecutar ocamlformatesta salida.

Comunidad
foro: https://discuss.ocaml.org/tags/ocamlformat
github: https://github.com/ocaml-ppx/ocamlformat
problemas: https://github.com/ocaml-ppx/ocamlformat/issues
Vea CONTRIBUYENDO para saber cómo ayudar.

Licencia
OCamlFormat tiene licencia MIT .
