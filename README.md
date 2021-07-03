# Herramienta de migraciones vivo
Esta herramienta esta diseñada para **estandarizar, guardar y facilitar** la ejecución de las **migraciones** en los diferentes ambientes de desarrollo.
## ⚙️ Instalación
Para poder correr las migraciones o crear nuevas, debemos instalar las dependencias del proyecto.
```sh
npm install
```

## ⚙️ Configuración
Dentro del proyecto encontraremos un archivo 📄 **/migrations_chiper/config/config.json** donde podremos configurar cada uno de los ambientes.
**NOTA:** Como en este archivo van datos sensibles, se puede combinar el proyecto con la dependencia **.env** para no subir los datos sensibles.

## 🔨︁ Crear una nueva migración
Mediante la terminal a la altura del proyecto podemos crear nuestras migraciones:
### Crear una nueva tabla
```sh
npx sequelize-cli model:generate --name <tableName> --attributes <tableCols>
```
Donde **<tableName>** es el nombre de la tabla y **<tableCols>** son las diferentes columnas.
Por ejemplo:
```
npx sequelize-cli model:generate --name UserAddress --attributes firstName:string,lastName:string,email:string
```
Una vez ejecutando este comando, se nos creará un archivo en la dirección 📁 **/migrations_chiper/migrations** con el contenido editable de nuestra migración. En la dirección 📁 **/migrations_chiper/models** se creará el modelo para usarse desde **JS** sin embargo, para efectos de este caso, no tiene reelevancia.

Ejemplo de migración creada en la carpeta 📁 **/migrations_chiper/migrations**:
```js
'use strict';
/**
 * Basic migration
 */
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      firstName: {
        type: Sequelize.STRING
      },
      lastName: {
        type: Sequelize.STRING(100)
      },
      balance: {
        type: Sequelize.FLOAT
      },
      admin: {
        type: Sequelize.BOOLEAN,
        defaultValue: false,
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      deletedAt: {
        allowNull: true,
        type: Sequelize.DATE
      },
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Users');
  }
};
```

### Crear una tabla relacionada (foreign key)
Siguiendo los mismos pasos de la sección anterior, dentro del archivo creado en 📁 **/migrations_chiper/migrations** agregaremos nuestra relación.
Por ejemplo:
```js
'use strict';
/**
 * Migration with foreing key
 * UserComments.userId -> Users.id
 */
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('UserComments', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      comment: {
        type: Sequelize.TEXT,
        allowNull: false
      },
      userId: {
        type: Sequelize.INTEGER,
        references: { model: 'Users', key: 'id' }
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('UserComments');
  }
};
```

### Crear una tabla y agregar indice a una columna
Para crear una tabla con un indice, es necesario crear la migración en base a una transacción, el motivo de esto es que si falla el anexo del indice a la columna, podemos crear un **Rollback** para evitar que la tabla se cree sin su respectivo indice.
Por ejemplo:
```js
'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.createTable('UserAddresses', {
        id: {
          allowNull: false,
          autoIncrement: true,
          primaryKey: true,
          type: Sequelize.INTEGER
        },
        address: {
          type: Sequelize.STRING
        },
        userId: {
          type: Sequelize.INTEGER,
          references: { model: 'Users', key: 'id' }
        },
        createdAt: {
          allowNull: false,
          type: Sequelize.DATE
        },
        updatedAt: {
          allowNull: false,
          type: Sequelize.DATE
        },
        deletedAt: {
          allowNull: true,
          type: Sequelize.DATE
        }
      }, { transaction });
      await queryInterface.addIndex('UserAddresses', ['userId', 'address'], { transaction });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('UserAddresses');
  }
};
```

## 🔨 Seeders (almacenar registros)
Para crear un seed hay que ejecutar en la línea de comandos:
```
npx sequelize-cli seed:generate --name demo-user
```
Este comando nos creará un archivo en 📁 **/migrations_chiper/seeders** que será una plantilla para ejecutar nuestros registros.
Por ejemplo:
```js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    /**
     * Add seed commands here.
     *
     * Example:
     * await queryInterface.bulkInsert('People', [{
     *   name: 'John Doe',
     *   isBetaMember: false
     * }], {});
    */
  },

  down: async (queryInterface, Sequelize) => {
    /**
     * Add commands to revert seed here.
     *
     * Example:
     * await queryInterface.bulkDelete('People', null, {});
     */
  }
};

```

## 🏃︁ Correr las migraciones
Para correr las migraciones alojadas en 📁 **/migrations_chiper/migrations** basta con correr el comando:
```
npx sequelize-cli db:migrate --env <enviroment>
```
Donde **<enviroment>** es el ambiente previamente configurado en el archivo 📄︁ **/migrations_chiper/config/config.json**
Este comando solo corre las migraciones pendientes, ya que **Sequelize** almacena un metadata que ayuda a identificar las migraciones ya realizadas previamente.

## Revertir migraciones
### Revertir la última migración
```
npx sequelize-cli db:migrate:undo
```
### Revertir todas las migraciones
```
npx sequelize-cli db:migrate:undo:all
```
### Revertir todas las migraciones hasta una migración en especifico
```
npx sequelize-cli db:migrate:undo:all --to XXXXXXXXXXXXXX-create-posts.js
```
## 🏃︁ Correr los seeders
Para correr todos los seeders pendientes basta con ejecutar:
```
npx sequelize-cli db:seed:all
```
### Revertir último seed
```
npx sequelize-cli db:seed:undo
```

### Revertir un seed especifico
```
npx sequelize-cli db:seed:undo --seed <name-of-seed-as-in-data>
```

### Revertir todos los seeds
```
npx sequelize-cli db:seed:undo:all
```

## 📄 NOTA
Cada comando para correr o revertir migraciones o seeders puede ir acompañado por la opción **--env <environment>** de lo contrario ejecutará el environment default **development**.

# 📄 Documentación

- [Migraciones y seeders](https://sequelize.org/master/manual/migrations.html)
- [Sequelize dataTypes](https://sequelize.org/v5/manual/data-types.html)