# #############################################################################
# https://gist.github.com/Guhan-SenSam/9dfb11b7bfd8fd24561f4fcd9ff0d5de
# #############################################################################

Métodos para optimizar el rendimiento de Kivy
=============================================

# A) info.md
============

Muchas personas afirman que kivy es lento. Si bien esto puede ser cierto, se debe principalmente a que Python es lento para ejecutarse en dispositivos Android. Por lo tanto, está en manos del programador optimizar adecuadamente su código para crear una aplicación de alto rendimiento.

La mayor parte del retraso en los dispositivos Android que ejecutan aplicaciones kivy surge debido a la creación de widgets. La creación de widgets sigue siendo el paso más lento en una aplicación kivy. Estos son algunos de los métodos que sigo para optimizar mis aplicaciones y asegurarme de que puedo alcanzar los 60 fps incluso en dispositivos antiguos.

Métodos de optimización:
========================

# B) precarga.md
================

1. Cargue todos sus widgets al inicio

Un método muy simple para mejorar el rendimiento de su aplicación es cargar todos los widgets pesados ​​o incluso todos los widgets de sus aplicaciones al inicio de su aplicación. Sin embargo, esto tiene el inconveniente de causar tiempos de carga más largos para su aplicación.

La forma de encubrir estos largos tiempos de carga sería también crear una pantalla de inicio de aplicación interesante con algunas animaciones. Buildozer admite animaciones de lottie (https://github.com/tshirtman/p4a_lottie_demo Cortesía de tshirtman).

Cargar todos sus widgets al inicio significa que simplemente puede agregar sus widgets a las pantallas a través de su código y el impacto en el rendimiento sería insignificante.

# C) animaciones.md
===================

2. Usa animaciones

Si necesita crear sus widgets durante el tiempo de ejecución del programa, puede usar animaciones para ocultar el retraso. Las animaciones ubicadas correctamente que animan la opacidad se pueden usar para ocultar el tiempo requerido para cargar widgets. El usuario percibiría esto solo como una transición y no como un retraso.

    Primero crea tu widget y dale una opacidad de cero
    Agregue el widget al diseño en el que debe estar
    Agregue una animación para la opacidad de ese widget. Establezca la duración en algo pequeño como 0,2 segundos.

Puede llevar esto más lejos si tiene que cargar varias instancias del mismo widget. En lugar de cargarlos todos juntos. Cargue un widget y en su animación complete la carga del siguiente. Mantenga la duración de la animación corta para que no se sumen y se vuelvan demasiado largas.

# D) Animaciones.py
===================

```
"""
Cargar multiples instancias de un widget durante un período de tiempo para evitar retrasos.
Útil para widgets más complejos que solo un botón.
"""

from kivy.app import App
from kivy.lang import Builder
from kivy.clock import Clock
from kivy.uix.button import Button
from kivy.animation import Animation


KV = '''
GridLayout:
    id: container
    cols: 5
    rows: 20
'''


class MessengerApp(App):

    counter = 0

    def build(self):
        self.kv = Builder.load_string(KV)
        return self.kv

    def on_start(self):
        Clock.schedule_interval(self.btn_create, 0.3)

    def btn_create(self,time):
        if self.counter < 50:
            btn = Button(text = str(self.counter))
            btn.opacity = 0                                   # Establece la opacidad del botón a 0
            self.kv.add_widget(btn)                           # Agregar el botón
            Animation(opacity = 1, duration = .25).start(btn) # La duración es < que la duración del reloj
            self.counter += 1


if __name__ == '__main__':
    MessengerApp().run()
```

# E) producerconsumer.md
========================

3. Uso de un modelo de productor consumidor para cargar widgets

Por defecto cuando creas un widget. Kivy creará el widget completo dentro de un solo marco. Pero si su widget es complejo y tiene numerosos widgets y diseños dentro, esto puede causar retrasos. Esto se debe a que kivy se abstendrá de mostrar ese marco hasta que el widget se haya creado por completo, lo que le parecerá a su usuario como un programa congelado o colgado.

En ciertos casos, usar una barra de carga para representar que la aplicación aún se está ejecutando, pero solo cargar datos es útil pero no agradable. Hace que el usuario espere a que la aplicación complete su operación, lo que puede ser frustrante. Tiene que haber algún tipo de acción visual.

En su lugar, podría cargar lentamente instancias de sus widgets en varios marcos. Básicamente, el código funciona de una manera en la que crearía un widget en un marco y luego la siguiente instancia en el siguiente marco y así sucesivamente. Por lo tanto, se pueden cargar grandes cantidades de datos rápidamente.

Consulte este enlace para obtener más información (https://blog.kivy.org/2014/01/producerconsumer-model-in-kivy/)

Nota: este método solo es efectivo si tiene que cargar una gran cantidad de widgets simples. Si tiene que cargar widgets complejos, sugeriría usar animaciones como en el método 2. O bien, podría dividir su widget complejo en otros más pequeños y simples y cargarlos en varios cuadros.

# F) simplificar.md
===================

4. Simplifique sus widgets tanto como sea posible

Esta es una tarea realmente dolorosa de hacer, pero definitivamente vale la pena. Si puede eliminar diseños y widgets innecesarios, puede mejorar significativamente el rendimiento de sus aplicaciones.

Algunos consejos:

    Use Boxlayouts y GridLayouts, ya que encontré que estos diseños pueden ayudar a posicionar su widget de manera más eficiente que los diseños flotantes.
    Unir etiquetas juntas. En lugar de tener tres widgets de etiquetas. Únalos para formar una sola etiqueta y use el marcado para modificar el estilo/color de fuente y cualquier otra propiedad.

```
text_to = '''[font=Roboto-Black.ttf][size=18sp][color=' +self.rgb2hex(self.text_color) + ']' + 
a[0] + 
'[/color][/size][/font]' + 
'\n' + 
'[font=Roboto-Regular.ttf][size=16sp][color=' +self.rgb2hex(self.secondary_text_color) +']' + 
a[1] + 
'[/color][/size][/font]'''
```

Este es un ejemplo de la combinación de dos etiquetas completamente diferentes en un solo objeto de etiqueta. Paso el texto como esta cadena y al mostrarlo aparecería como si fueran etiquetas separadas, pero en realidad son una sola etiqueta. En algunos casos, esto puede reducir a la mitad la cantidad de widgets a crear.

# G) usarhilos.md
=================

5. Use Threading siempre que sea posible

No hay forma de actualizar la interfaz de usuario o crear widgets en un hilo, ya que kivy no garantiza un comportamiento estable de esta manera. En su lugar, ejecute su interfaz de usuario en su subproceso principal y ejecute sus otras funciones de aplicación en un subproceso secundario. Esto, si se hace correctamente y para el caso de uso correcto, puede ser muy eficaz para mejorar el rendimiento.

Consulte aquí para obtener información sobre el enhebrado de python de una manera sencilla: (https://www.geeksforgeeks.org/multithreading-python-set-1/)

# H) LazyLoading.md
===================

6. pantallas de carga perezosas

Este método se puede usar para aumentar significativamente los tiempos de carga de su aplicación en Android. Funciona cargando solo la pantalla principal y luego cargando lentamente las otras pantallas de su aplicación con el tiempo. El código es de @Kulothungan16 ... Consulte su repositorio para obtener más información (https://github.com/Kulothungan16/kivy-lazy-loading-template)

# I) Misc.md
============

7. cosas misceláneas

    Reduzca sus declaraciones de importación (mejora la hora de inicio pero no mucho).

Incorrecto:

import kivy

Correcto:

from kivy import # Lo que necesites importar

    Sea más inteligente con la colocación de widgets. Intente agrupar los widgets en una sola instancia. No tenga miedo de jugar con el código fuente de kivy o de crear sus widgets personalizados más eficientes. Recuerde que la biblioteca kivy debe seguir siendo personalizable, por lo que solo se puede eliminar y optimizar hasta cierto punto. Pero si crea sus propios widgets personalizados, puede eliminar el código innecesario que podría acelerar la creación de widgets.
    No use demasiadas instancias de animación ya que esto es lento en Android
    Reduzca el valor del paso de la animación a 1/30 para que la animación solo se actualice cada dos fotogramas. Útil especialmente para la animación de opacidad ya que la diferencia no es visible en absoluto. Puede ir tan bajo como 1/10 para animaciones basadas en opacidad.

Animation(opacity = 1, d = .3, step = 1/30)

    Intente realizar una carga previa de widgets en segundo plano, como cuando el usuario no toca la pantalla durante un cierto período de tiempo. Luego puede detener esta carga en el próximo evento de toque, etc. Pero recuerde que no debe cargar mucho en segundo plano, ya que esto puede causar una gran cantidad de uso de memoria y podría resultar en una experiencia de usuario aún más lenta.
    Reutilice los widgets tanto como pueda. En lugar de recrear un widget cada vez que algún valor cambia, reutilice los ya creados y solo cambie los valores necesarios.
