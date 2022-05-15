**AWS CloudFormation**

1. ¿Qué es CloudFormation?
   1) Conceptos de CloudFormation
   1) ¿Cómo funciona CloudFormation
1. Seguridad en AWS CloudFormation
   1) Protección de datos
   1) Controlar el acceso con AWS IAM
   1) Registro de llamadas a la API
   1) Seguridad de la infraestructura
   1) Resiliencia
   1) Prácticas de seguridad recomendadas

**¿Qué es AWS CloudFormation?**

Es un servicio que lo ayuda a modelar y configurar sus recursos de AWS, por lo que podrá dedicar menos tiempo a la administración de dichos recursos y más tiempo a centrarse en las aplicaciones que se ejecutan en AWS. Puede crear una plantilla que describa todos los recursos de AWS que desea (como instancias Amazon EC2 o instancias de base de datos de Amazon RDS) y CloudFormation se encargará del aprovisionamiento y la configuración de dichos recursos. No es necesario crear y configurar individualmente los recursos AWS ni averiguar qué depende de qué; CloudFormation se encarga de todo eso. Las siguientes situaciones demuestran cómo puede ayudarle CloudFormation.

**Conceptos de AWS CloudFormation**

Al utilizar AWS CloudFormation, trabaje con plantillas y pilas. Puede crear plantillas para describir los recursos de AWS y sus propiedades. Cuando se crea una pila, CloudFormation aprovisiona los recursos que se describen en su plantilla. Cuando se crea una pila, debe especificarse una plantilla que CloudFormation utiliza para crear lo que se describe en la plantilla.

**Plantillas:**

Es un archivo de texto con formato JSON o YAML. Estos se pueden guardar con extensiones como. json, .yaml, .template  o .txt. Cloudformation utiliza estas plantillas como planos para la creación de los recursos de AWS. Por ejemplo, en una plantilla puede describir una instancia de Amazon EC2, como el tipo de instancia, el ID de AMI, mapeos de dispositivos de bloques y e nombre de su par de claves de EC2. Cuando se crea una pila, debe especificarse una plantilla que CloudFormation utiliza para crear lo que se describe en la plantilla.

Por ejemplo, si creó una pila con la siguiente plantilla, CloudFormation aprovisiona una instancia con un ID de AMI ami-0ff8a91507f77f867, un tipo de instancia t2.micro, el nombre del par de claves testkey y un volumen de Amazon EBS.

**Ej. JSON**

    {
      "AWSTemplateFormatVersion" : "2010-09-09",
      "Description" : "A sample template",
      "Resources" : {
        "MyEC2Instance" : {
          "Type" : "AWS::EC2::Instance",
          "Properties" : {
            "ImageId" : "ami-0ff8a91507f77f867",
            "InstanceType" : "t2.micro",
            "KeyName" : "testkey",
            "BlockDeviceMappings" : [
              {
                "DeviceName" : "/dev/sdm",
                "Ebs" : {
                  "VolumeType" : "io1",
                  "Iops" : 200,
                  "DeleteOnTermination" : false,
                  "VolumeSize" : 20
                }
              }
            ]
          }
        }
      }
    }


**Ej. YAML**

    AWSTemplateFormatVersion: "2010-09-09"
    Description: A sample template
    Resources:
    MyEC2Instance:
        Type: "AWS::EC2::Instance"
        Properties: 
        ImageId: "ami-0ff8a91507f77f867"
        InstanceType: t2.micro
        KeyName: testkey
        BlockDeviceMappings:
            -
            DeviceName: /dev/sdm
            Ebs:
                VolumeType: io1
                Iops: 200
                DeleteOnTermination: false
                VolumeSize: 20


También puede especificar varios recursos en una única plantilla y configurar dichos recursos para que funcionen conjuntamente. Por ejemplo, puede modificar la plantilla anterior para que incluya una dirección IP elástica (EIP) y asociarla a la instancia Amazon EC2, tal y como se muestra en el ejemplo siguiente:

**Ej. JSON**

    {
        "AWSTemplateFormatVersion" : "2010-09-09",
        "Description" : "A sample template",
        "Resources" : {
            "MyEC2Instance" : {
                "Type" : "AWS::EC2::Instance",
                "Properties" : {
                    "ImageId" : "ami-0ff8a91507f77f867",
                    "InstanceType" : "t2.micro",
                    "KeyName" : "testkey",
                    "BlockDeviceMappings" : [
                        {
                        "DeviceName" : "/dev/sdm",
                        "Ebs" : {
                            "VolumeType" : "io1",
                            "Iops" : 200,
                            "DeleteOnTermination" : false,
                            "VolumeSize" : 20
                        }
                        }
                    ]
                }
            },
            "MyEIP" : {
                "Type" : "AWS::EC2::EIP",
                "Properties" : {
                "InstanceId" : {"Ref": "MyEC2Instance"}
                }
            }
        }
    }


**Ej. YAML**

    AWSTemplateFormatVersion: "2010-09-09"
    Description: A sample template
    Resources:
    MyEC2Instance:
        Type: "AWS::EC2::Instance"
        Properties: 
        ImageId: "ami-0ff8a91507f77f867"
        InstanceType: t2.micro
        KeyName: testkey
        BlockDeviceMappings:
            -
            DeviceName: /dev/sdm
            Ebs:
                VolumeType: io1
                Iops: 200
                DeleteOnTermination: false
                VolumeSize: 20
    MyEIP:
        Type: AWS::EC2::EIP
        Properties:
        InstanceId: !Ref MyEC2Instance


Las plantillas anteriores se centraron en una única instancia Amazon EC2; sin embargo, las plantillas de CloudFormation tienen capacidades adicionales que puede utilizar para crear conjuntos complejos de recursos y reutilizar esas plantillas en múltiples contextos. Por ejemplo, puede agregar parámetros de entrada cuyos valores se especifican al crear una pila de CloudFormation. En otras palabras, puede especificar un valor como el tipo de instancia al crear una pila en lugar de al crear la plantilla, lo que hace a la plantilla más fácil de reutilizar en diferentes situaciones.

**Pilas:**

Al utilizar CloudFormation, los recursos relacionados se administran como una sola unidad, denominada pila. Puede crear, actualizar y eliminar una colección de recursos mediante la creación, actualización y eliminación de pilas. Todos los recursos de una pila se definen mediante la plantilla de CloudFormation de la pila. Supongamos que ha creado una plantilla que incluye un grupo de Auto Scaling, un balanceador de carga Elastic Load Balancing y una instancia de base de datos Amazon Relational Database Service (Amazon RDS). Para crear esos recursos, debe crear una pila enviando la plantilla que ha creado y CloudFormation aprovisiona todos esos recursos para usted. También puede trabajar con pilas mediante la consola de CloudFormation, la API o la AWS CLI.

**Conjuntos de cambios**

Si necesita realizar cambios en los recursos que están ejecutándose en una pila, actualice la pila. Antes de realizar cambios en los recursos, puede generar un conjunto de cambios, que es un resumen de los cambios propuestos. Los conjuntos de cambios le permiten ver cómo afectan los cambios a sus recursos en ejecución, en especial para los recursos críticos, antes de implementarlos.

Por ejemplo, si cambia el nombre de una instancia de base de datos Amazon RDS, CloudFormation creará una nueva base de datos y eliminará la antigua. Se perderán los datos de la antigua base de datos a menos que ya haya hecho una copia de seguridad. Si genera un cambio conjunto, verá que el cambio provocará la sustitución de la base de datos y usted podrá planificar en consecuencia antes de actualizar la pila.
# **¿Cómo funciona AWS CloudFormation?**

AWS CloudFormation realiza llamadas de servicio subyacentes a AWS para aprovisionar y configurar sus recursos. CloudFormation sólo puede realizar acciones para las que tenga permiso.

Su plantilla declara todas las llamadas que CloudFormation hace. Por ejemplo, suponga que tiene una plantilla que describe una instancia EC2 con un tipo de instancia t2.micro. Cuando utiliza la plantilla para crear una pila, CloudFormation llama a la API de creación de instancias Amazon EC2 y especifica el tipo de instancia como t2.micro. En el siguiente diagrama se resume el flujo de trabajo de CloudFormation para la creación de pilas.

![](./img/img1.png)

1. Para crear o modificar una plantilla de CLoudFormation en formato JSON o YAML es posible usar el AWS CloudFormation Designer o bien sea el editor de texto de su preferencia. También puede optar por utilizar plantillas proporcionada. La plantilla de CloudFormation describe los recursos que desea y su configuración. Por ejemplo, suponga que desea crear una instancia EC2. La plantilla puede indicar una Amazon EC2 instancia EC2 y describir sus propiedades, tal y como se muestra en el ejemplo siguiente:

**JSON:**

    {
        "AWSTemplateFormatVersion" : "2010-09-09",
        "Description" : "A simple EC2 instance",
        "Resources" : {
                "MyEC2Instance" : {
                "Type" : "AWS::EC2::Instance",
                "Properties" : {
                    "ImageId" : "ami-0ff8a91507f77f867",
                    "InstanceType" : "t2.micro"
                }
            }
        }
    }


**YAML:**

    AWSTemplateFormatVersion: '2010-09-09'
    Description: A simple EC2 instance
    Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        Properties:
        ImageId: ami-0ff8a91507f77f867
        InstanceType: t2.micro


1. Guarde la plantilla localmente o en un bucket de S3. Si creó una plantilla, guárdela con un archivo de extensión como .json, .yaml o .txt.

1. Cree una pila de CloudFormation al especificar la ubicación del archivo de plantilla, como, por ejemplo, una ruta en el equipo local o una URL de Amazon S3. Si la plantilla contiene parámetros, puede especificar los valores de entrada al crear la pila. Los parámetros le permiten pasar valores a la plantilla para poder personalizar sus recursos cada vez que crea una pila.
- Puede crear pilas utilizando la consola de CloudFormation, la API o la AWS CLI.
- CloudFormation aprovisiona y configura recursos realizando llamadas a los servicios de AWS que se describen en su plantilla.
- Una vez que se han creado todos los recursos, CloudFormation informa que se ha creado su pila. A continuación, puede comenzar a utilizar los recursos de la pila. Si se produce un error durante la creación de la pila, CloudFormation revierte los cambios eliminando los recursos que creó.

**Actualización de una pila con conjuntos de cambios**

Cuando se requiera actualizar los recursos de su pila, es posible modificar la plantilla de los recursos. Para actualizar una pila, cree un conjunto de cambios enviando una versión modificada de la plantilla de pila original, diferentes valores de parámetros de entrada, o ambos. CloudFormation compara la plantilla modificada con la plantilla original y genera un conjunto de cambios. El conjunto de cambios enumera las modificaciones propuestas. Tras revisar los cambios, puede iniciar el conjunto de cambios para actualizar la pila o puede crear un conjunto de cambios nuevo. En el siguiente diagrama se resume el flujo de trabajo para la actualización de una pila. **Es importante tener en cuenta que las actualizaciones pueden causar interrupciones. Dependiendo del recurso y de las propiedades que esté actualizando, una actualización podría interrumpir o incluso sustituir un recurso existente. Para obtener más información, consulte Actualizaciones de pila de AWS CloudFormation.**

![](./img/img2.png)
1. Puede modificar una plantilla de pila de CloudFormation utilizando AWS CloudFormation Designer o un editor de texto. Por ejemplo, si desea cambiar el tipo de instancia de una instancia EC2, deberá cambiar el valor de la propiedad InstanceType en la plantilla original de la pila.

1. Guarde la plantilla de CloudFormation localmente o en un bucket de S3.

1. Cree un conjunto de cambios especificando la pila que desea actualizar y la ubicación de la plantilla modificada, como por ejemplo una ruta en su equipo local o una dirección URL de Amazon S3. Si la plantilla contiene parámetros, puede especificar valores al crear el conjunto de cambios.

1. Vea el conjunto de cambios para comprobar que CloudFormation realizará los cambios que espera. Por ejemplo, compruebe si CloudFormation reemplazará cualquier recurso crítico de la pila. Puede crear tantos conjuntos de cambios como necesite hasta que haya incluido los cambios que desea. **Cabe destacar que los conjuntos de cambios no indican si la actualización de la pila se realizará correctamente. Por ejemplo, un conjunto de cambios no verifica si se sobrepasará una cuota de cuenta, si va a actualizar un recurso que no admite actualizaciones o si no tiene suficientes permisos para modificar un recurso, puede provocar una actualización de pila que fallará.**

1. Inicie el conjunto de cambios que desea aplicar a la pila. CloudFormation actualiza la pila al actualizar solo los recursos que ha modificado e indica que la pila se ha actualizado correctamente. Si se produce un error en la actualización de la pila, CloudFormation revierte los cambios para restaurar la pila al último estado de funcionamiento conocido.

**Eliminación de una pila**

Para eliminar una pila es requerido especificar dicha pila a eliminar, y CloudFormation eliminara la pila y todos sus recursos, es posible eliminar pilas utilizando la consola de CloudFormation, la API o la AWS CLI.

Si desea eliminar una pila, pero desea conservar algunos recursos de la pila, puede usar una política de eliminación para conservar dichos recursos.

- **Atributo DeletionPolicy:** Con el atributo DeletionPolicy puede conservar y, en algunos casos, crear un backup de un recurso cuando se elimina su pila. Puede especificar un atributo DeletionPolicy para cada recurso que desea controlar. Si un recurso no tiene ningún atributo DeletionPolicy, AWS CloudFormation elimina el recurso de forma predeterminada.

Esta capacidad también se aplica a las operaciones de actualización de pilas que hacen que se eliminen recursos de las pilas. Por ejemplo, si elimina el recurso de la plantilla de pila y, a continuación, actualiza la pila con la plantilla. Esta capacidad no se aplica a los recursos cuya instancia física se sustituye durante las operaciones de actualización de la pila. Por ejemplo, si edita las propiedades de un recurso de tal forma que CloudFormation sustituya dicho recurso durante la actualización de una pila.

Una vez que se han eliminado todos los recursos, CloudFormation señala que se ha eliminado correctamente su pila. Si CloudFormation no puede eliminar un recurso, no se eliminará la pila. Cualquier recurso que no se haya eliminado permanecerá hasta que pueda eliminar correctamente la pila.


**Seguridad en AWS CloudFormation**

La seguridad en la nube de AWS es la mayor prioridad. Como cliente de AWS, se beneficiará de una arquitectura de red y de centros de datos diseñados para satisfacer los requisitos de seguridad de las organizaciones más exigentes.

La seguridad es una responsabilidad compartida entre AWS y usted. El modelo de responsabilidad compartida la describe como seguridad de la nube y seguridad en la nube:

- **Seguridad de la nube** – AWS es responsable de proteger la infraestructura que ejecuta servicios de AWS en la nube de AWS. AWS también proporciona servicios que puede utilizar de forma segura. 
- **Seguridad en la nube** – Su responsabilidad viene determinada por el servicio de AWS que utilice. También es responsable de otros factores, incluida la confidencialidad de los datos, los requisitos de la empresa y la legislación y los reglamentos aplicables.


**Protección de datos en AWS CloudFormation**

AWS es responsable de proteger la infraestructura global que ejecuta todo Nube de AWS. Usted es responsable de mantener el control sobre el contenido alojado en esta infraestructura. Este contenido incluye la configuración de seguridad y las tareas de administración de los servicios de AWS que utiliza.

Para fines de protección de datos, se recomienda proteger las credenciales de Cuenta de AWS y configurar cuentas de usuario individuales con AWS Identity and Access Management (IAM). De esta manera, solo se otorgan a cada usuario los permisos necesarios para cumplir con sus obligaciones laborales. También le recomendamos proteger sus datos de las siguientes formas:

- Utilice la autenticación multifactor (MFA) con cada cuenta (Aporta seguridad adicional, ya que exige a los usuarios que proporcionen una autenticación exclusiva obtenida de un mecanismo de MFA admitido por AWS, además de sus credenciales de inicio de sesión habituales, para obtener acceso a los sitios web o servicios de AWS).
- Utilice SSL/TLS para comunicarse con los recursos de AWS (La tecnología estándar para mantener segura una conexión a Internet, así como para proteger cualquier información confidencial que se envía entre dos sistemas e impedir que los delincuentes lean y modifiquen cualquier dato que se transfiera). 
- Configure la API y el registro de actividad del usuario con AWS CloudTrail (Las acciones que realiza un usuario, rol o servicio de AWS se registran como eventos en CloudTrail).
- Utilice las soluciones de cifrado de AWS, junto con todos los controles de seguridad predeterminados dentro de los servicios de AWS.


**Controlar el acceso con AWS Identity and Access Management**

Puede crear usuarios de IAM para controlar quién tiene acceso a qué recursos en su cuenta de AWS. Puede utilizar IAM con AWS CloudFormation para controlar lo que los usuarios pueden hacer con AWS CloudFormation, como, por ejemplo, si pueden ver las plantillas de pila, crear pilas o eliminar pilas.

Además de acciones de AWS CloudFormation, puede administrar qué servicios y recursos de AWS están disponibles para cada usuario. De esa manera, puede controlar a qué recursos pueden obtener acceso los usuarios cuando utilizan AWS CloudFormation. Por ejemplo, puede especificar qué usuarios pueden crear instancias Amazon EC2, terminar instancias de base de datos o actualizar VPC. Estos mismos permisos se aplican siempre que utilizan AWS CloudFormation para realizar esas acciones.


**Registro de llamadas a la API de AWS CloudFormation con AWS CloudTrail**

AWS CloudFormation está integrado con AWS CloudTrail, un servicio que proporciona un registro de las acciones de un usuario, un rol, o un servicio de AWS en CloudFormation. CloudTrail captura todas las llamadas a la API de CloudFormation como eventos, incluidas las llamadas procedentes de la consola de CloudFormation y de las llamadas del código a las API de CloudFormation. Si crea un registro de seguimiento, puede habilitar la entrega continua de eventos de CloudTrail a un bucket de Amazon S3, incluidos los eventos de CloudFormation. Si no configura un registro de seguimiento, puede ver los eventos más recientes en la consola de CloudTrail en el Event history (Historial de eventos). Mediante la información que recopila por CloudTrail, se puede determinar la solicitud que se envió a CloudFormation, la dirección IP desde la que se realizó la solicitud, quién la realizó, cuándo se realizó y detalles adicionales.


**Seguridad de la infraestructura en AWS CloudFormation**

Como servicio administrado, AWS CloudFormation está protegido por los procedimientos de seguridad de red globales de AWS, donde puede utilizar llamadas a la API publicadas en AWS para acceder a AWS CloudFormation a través de la red. 

Puede llamar a estas operaciones de la API desde cualquier ubicación de red, pero AWS CloudFormation admite políticas de acceso basadas en recursos, que pueden incluir restricciones en función de la dirección IP de origen. También puede utilizar políticas de AWS CloudFormation para controlar el acceso desde puntos de enlace específicos de Amazon Virtual Private Cloud (Amazon VPC) o VPC específicas. Este proceso aísla con eficacia el acceso de red a un recurso de AWS CloudFormation determinado únicamente desde la VPC específica de la red de AWS.


**Resiliencia en AWS CloudFormation**

La infraestructura global de AWS está conformada por regiones y zonas de disponibilidad de AWS. Las regiones de AWS proporcionan varias zonas de disponibilidad físicamente independientes y aisladas que se encuentran conectadas mediante redes con un alto nivel de rendimiento y redundancia, además de baja latencia. Con las zonas de disponibilidad, puede diseñar y utilizar aplicaciones y bases de datos que realizan una conmutación por error automática entre las zonas sin interrupciones. Las zonas de disponibilidad tienen una mayor disponibilidad, tolerancia a errores y escalabilidad que las infraestructuras tradicionales de centros de datos únicos o múltiples.

**Prácticas de seguridad recomendadas para AWS CloudFormation**

AWS CloudFormation proporciona un número de características de seguridad que debe tener en cuenta a la hora de desarrollar e implementar sus propias políticas de seguridad. Las siguientes prácticas recomendadas son directrices generales y no suponen una solución de seguridad completa. Puesto que es posible que estas prácticas recomendadas no sean adecuadas o suficientes para el entorno, considérelas como consideraciones útiles en lugar de como normas.

- Utilice IAM para controlar el acceso: IAM es un servicio AWS que puede utilizar para administrar los usuarios y sus permisos en AWS. Puede utilizar IAM con AWS CloudFormation para especificar qué acciones de AWS CloudFormation pueden realizar los usuarios, como ver plantillas de pila, crear pilas o eliminar pilas. Además, cualquiera que administre pilas de AWS CloudFormation necesitará permisos para acceder a los recursos de dichas pilas. Por ejemplo, si los usuarios desean usar AWS CloudFormation para lanzar, actualizar o terminar instancias Amazon EC2, deben tener permiso para llamar a las acciones de Amazon EC2 relevantes.
- No integre credenciales en sus plantillas: En lugar de incrustar información confidencial en sus plantillas de AWS CloudFormation, le recomendamos que utilice referencias dinámicas en su plantilla de pila. Las referencias dinámicas son una forma eficaz y coherente de hacer referencia a valores externos almacenados y administrados en otros servicios, como el AWS Administrador de sistemas de Parameter Store AWSo Secrets Manager. Cuando se utiliza una referencia dinámica, CloudFormation recupera el valor de la referencia especificada cuando es necesario durante las operaciones de pila y de conjunto de cambios, y pasa el valor al recurso correspondiente. Sin embargo, CloudFormation nunca almacena el valor de referencia real. 
- Utilice AWS CloudTrail para registrar las llamadas de AWS CloudFormation: WS CloudTrail rastrea a cualquier persona que realice llamadas a la API de AWS CloudFormation en una cuenta de AWS. Las llamadas a la API se registran cuando se utiliza la API de AWS CloudFormation, la consola de AWS CloudFormation, una consola de back-end o comandos AWS CloudFormation AWS CLI. Active el registro y especifique un bucket de Amazon S3 para almacenar los registros.

