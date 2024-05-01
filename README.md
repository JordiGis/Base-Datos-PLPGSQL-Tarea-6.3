# Funciones en PostgreSQL
## Realiza las funciones suma y divide en PostgreSQL.

```SQL
    -- Función que suma dos números enteros
    CREATE OR REPLACE
    FUNCTION testPostgre.suma(IN num1 INT, IN num2 INT) RETURNS INT
    LANGUAGE plpgsql
        AS $function$
            DECLARE
                resultado INT;
            BEGIN
                resultado := num1 + num2;
                RETURN resultado;
            END
        $function$
    
    -- Función que divide dos números enteros
    CREATE OR REPLACE
    FUNCTION testPostgre.divide(IN num1 INT, IN num2 INT) RETURNS INT
    LANGUAGE plpgsql
        AS $function$
            DECLARE
                resultado INT;
            BEGIN
                resultado := num1 / num2;
                RETURN resultado;
            END
        $function$
```

## Función que devuelva la lista de facturas entre dos fechas.

```SQL
    CREATE OR REPLACE
    FUNCTION testPostgre.listaFacturasFechas(IN fechaIni DATE, IN fechaFin DATE) 
    RETURNS TABLE (
        id_factura INT,
        fecha_factura DATE,
        total_facturado FLOAT
    ) LANGUAGE plpgsql
        AS $function$
            BEGIN
                -- Por lo que he vsisto retunr query es para devolver una tabla, con diferentes columnas, asi dara el resultado de la consulta, para ello tambien haria falta especificar en Returns que tiene que dar una tabla, diciendole que columnas van a haber
                RETURN QUERY
                SELECT id_factura, fecha_factura, total_facturado
                FROM facturas
                WHERE fecha_factura BETWEEN fechaIni AND fechaFin;
            END
        $function$
```

## Función llamada Login que indique si un usuario es válido tras INTroducir su id y su contraseña.

```SQL
    CREATE OR REPLACE
    FUNCTION testPostgre.login(IN id_usuario INT, IN pass VARCHAR(100)) RETURNS BOOLEAN
    LANGUAGE plpgsql
        AS $function$
            DECLARE existe BOOLEAN;
            BEGIN
                SELECT EXISTS(
                    -- Existe solo comprueba si hay algo, asi que si dabuelve 1, es que hay algo, si no, no hay nada, asi hace que la consulta sea un poco mas rapida
                    SELECT 1
                    FROM usuarios
                    WHERE id_usuario = id_usuario AND pass = pass
                ) INTO existe;
                RETURN existe;
            END
        $function$
```

## Función que calcule la letra del DNI.

```SQL
    CREATE OR REPLACE
    FUNCTION testPostgre.letraDNI(IN dni INT) RETURNS CHAR 
    LANGUAGE plpgsql
        AS $function$
            DECLARE
                letrasDNI CHAR[] := '{T,R,W,A,G,M,Y,F,P,D,X,B,N,J,Z,S,Q,V,H,L,C,K,E}';
                letra CHAR;
            BEGIN
                letra := letrasDNI[dni % 23 + 1];
                RETURN letra;
            END
        $function$
    
```

## Una función que devuelva la concatenación de 2 cadenas que se le pasarán como parámetro.

```SQL
    CREATE OR REPLACE
    FUNCTION testPostgre.concatenarCadenas(IN cadena1 VARCHAR(100), IN cadena2 VARCHAR(100)) RETURNS VARCHAR(200) 
    LANGUAGE plpgsql
        AS $function$
            DECLARE
                resultado VARCHAR(200);
            BEGIN
                -- La concatenacion es rara, se hace con ||, no con +, aun que entiendo, porque se fueran numeros seria un lio, pero ya se pone arriva que son cadenas
                resultado := cadena1 || cadena2;
                RETURN resultado;
            END
        $function$
```
# Cursores PostgreSQL
## Incorpora FOR UPDATE/SHARE, realiza una actualización en el cursor y comprueba que ahora el cursor es sensitive.

```SQL
    CREATE OR REPLACE
    PROCEDURE empresa.cambioDeNombrePorID(id INT, nombre VARCHAR(100))
    LANGUAGE plpgsql
    AS $PROCEDURE$
    DECLARE cursor_usuarios CURSOR FOR
    SELECT * FROM empresa.usuarios
    WHERE id = id
    FOR UPDATE of nombre;
    -- la variable usuario alamacenara el registro de la tabla usuarios
    usuario empresa.usuarios%rowtype;

    BEGIN
    OPEN cursor_usuarios;
    FETCH cursor_usuarios INTO usuario;
    usuario.nombre = nombre;
    UPDATE empresa.usuarios SET nombre = usuario.nombre WHERE current of cursor_usuarios;
    CLOSE cursor_usuarios;
    END;
```

## Piensa en un caso en el que sea necesario utilizar un cursor. Intenta programarlo. 

En que caso se podria aplicar un cursor?
Un cursor se podria usar cuando se necesita recorrer una tabla de manera secuencial, por ejemplo para realizar una operacion en cada registro de la tabla, y no se quiere hacer directamente, para tener un mayor control sobre los registros que se estan modificando, por ejemplo cuando tenemos que hacer una operación compleja, que afecte a diversas tablas.

```SQL
    -- Este procedimiento tiene que cambiar el importe de todos los precio de los productos, y tiene que mostrar el precio anterior y el nuevo precio, para ello se usara un cursor, que tenga almacenado el precio anterior y el nuevo precio.
    CREATE OR REPLACE
    PROCEDURE empresa.cambioDePrecio()
    LANGUAGE plpgsql
    AS $PROCEDURE$
    DECLARE cursor_precios CURSOR FOR
    SELECT * FROM empresa.productos;
    producto empresa.productos%rowtype;

    BEGIN
    OPEN cursor_precios;
    LOOP
    FETCH cursor_precios INTO producto;
    exit WHEN NOT found;
    raise notice 'Precio anterior: %', producto.precio;
    producto.precio = producto.precio * 1.1;
    raise notice 'Nuevo precio: %', producto.precio;
    UPDATE empresa.productos SET precio = producto.precio WHERE current of cursor_precios;
    END LOOP;
    CLOSE cursor_precios;
    END;
```	

## Investiga si existe algún tipo de estructura de bucle alternativa que permita el recorrido de un cursor.

Como alternativa al bucle LOOP, se puede usar un bucle FOR, que es mas sencillo de usar, ya que no se necesita declarar una variable de control, y se puede usar directamente el cursor, su estructura es la siguiente:

```SQL
    FOR variable IN cursor_name LOOP
    -- Cosas que se quieran hacer
    END LOOP;
```

Aquí se puede ver un ejemplo de utilización, basado en el ejemeplo anterior.

```SQL
    CREATE OR REPLACE
    PROCEDURE empresa.cambioDePrecio()
    LANGUAGE plpgsql
    AS $PROCEDURE$
    DECLARE cursor_precios CURSOR FOR
    SELECT * FROM empresa.productos;
    producto empresa.productos%rowtype;

    BEGIN
    FOR producto IN cursor_precios LOOP
    raise notice 'Precio anterior: %', producto.precio;
    producto.precio = producto.precio * 1.1;
    raise notice 'Nuevo precio: %', producto.precio;
    UPDATE empresa.productos SET precio = producto.precio WHERE current of cursor_precios;
    END LOOP;
    CLOSE cursor_precios;
    END;
```

## Ejemplo

```SQL
    CREATE OR REPLACE
    PROCEDURE empresa.ejemplo_recorrido_cursor(prec numeric)
    LANGUAGE plpgsql
    AS $PROCEDURE$
    DECLARE  
    cursor_art cursor(precio_art integer) FOR
    SELECT id FROM empresa.articulos
    WHERE precio >= precio_art order by id;
    id_art integer;
    BEGIN
    
    OPEN cursor_art(prec);  
    FETCH cursor_art INTO id_art;   -- next (por defecto)
    raise notice 'Visitando articulo %',id_art;
    FETCH last FROM cursor_art INTO id_art;
    raise notice '(last) Visitando articulo %',id_art;
    FETCH first FROM cursor_art INTO id_art;
    raise notice '(first) Visitando articulo %',id_art;
    FETCH next FROM cursor_art INTO id_art;
    raise notice '(next) Visitando articulo %',id_art;
    FETCH prior FROM cursor_art INTO id_art;
    raise notice '(prior) Visitando articulo %',id_art;
    
    -- salto absoluto
    FETCH absolute 5 FROM cursor_art INTO id_art;
    raise notice '(absolute) Visitando articulo %',id_art;
    -- salto relativo
    FETCH relative 2 FROM cursor_art INTO id_art;
    raise notice '(relative +) Visitando articulo %',id_art;
    FETCH relative -2 FROM cursor_art INTO id_art;
    raise notice '(relative -) Visitando articulo %',id_art;

    -- uso de move ...
    move absolute 2 FROM cursor_art;
    if found then
    raise notice 'Encontrada posición 2'; 
    move prior FROM cursor_art;
    FETCH FROM cursor_art INTO id_art;
    raise notice 'Es el articulo %',id_art;
    else
    raise notice 'No existe';
    move first FROM cursor_art;
    END if;

    CLOSE cursor_art;

    END;
    $PROCEDURE$
```
