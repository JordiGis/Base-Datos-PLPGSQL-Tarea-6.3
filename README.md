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
