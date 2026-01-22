# 🛡️ 1838 - Administración de MySQL: Seguridad y Optimización

<div align="center">

**Curso Profesional de Administración de Bases de Datos MySQL**

[![MySQL Admin](https://img.shields.io/badge/MySQL-Administración-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Alura Latam](https://img.shields.io/badge/Plataforma-Alura_Latam-00C86F?style=for-the-badge)](https://www.aluracursos.com/)
[![Database Security](https://img.shields.io/badge/Seguridad-Bases_de_Datos-CC2927?style=for-the-badge)](https://www.w3schools.com/sql/)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-yellow?style=for-the-badge)](LICENSE)

[🔐 Seguridad](#-seguridad-y-autenticación) • 
[⚡ Optimización](#️-optimización-del-rendimiento) • 
[📊 Monitoreo](#-monitoreo-y-mantenimiento) • 
[💾 Backup](#-backup-y-recovery) • 
[👨‍💻 Autor](#-autor)

</div>

---

## 🎯 Descripción del Curso

Este repositorio contiene los **comandos, scripts y configuraciones** del curso **"Administración de MySQL: Seguridad y optimización de la base de datos"** de Alura Latam. Este curso te capacita para administrar profesionalmente servidores MySQL, implementando medidas de seguridad robustas y optimizando el rendimiento para entornos productivos.

### 🎓 Público Objetivo
- **Administradores de bases de datos** (DBAs) principiantes
- **Desarrolladores** que necesitan administrar sus propios entornos MySQL
- **DevOps** responsables de la infraestructura de bases de datos
- **Estudiantes** que buscan especializarse en administración de BD
- **Profesionales IT** que gestionan servicios MySQL en producción

### 🚨 Importancia de la Administración Profesional
| Área Crítica | Consecuencias de Mala Administración | Beneficios de Buena Administración |
|--------------|--------------------------------------|------------------------------------|
| **Seguridad** | Fugas de datos, ataques SQL injection | Datos protegidos, cumplimiento normativo |
| **Rendimiento** | Lentitud, timeout en aplicaciones | Respuestas rápidas, mejor experiencia usuario |
| **Disponibilidad** | Caídas del servicio, pérdida de negocio | Alta disponibilidad, business continuity |
| **Integridad** | Corrupción de datos, inconsistencias | Datos confiables, decisiones acertadas |

---

## 🔐 Seguridad y Autenticación

### **1. Gestión de Usuarios y Privilegios**
```sql
-- Crear usuario con password seguro
CREATE USER 'app_user'@'localhost' 
IDENTIFIED BY 'Str0ngP@ssw0rd!2024'
PASSWORD EXPIRE INTERVAL 90 DAY
FAILED_LOGIN_ATTEMPTS 5
PASSWORD_LOCK_TIME 1;

-- Privilegios granulares por objeto
GRANT SELECT, INSERT, UPDATE ON tienda.* TO 'app_user'@'localhost';
GRANT EXECUTE ON PROCEDURE tienda.sp_venta_mensual TO 'app_user'@'localhost';

-- Privilegios a nivel de columna
GRANT SELECT (nombre, email), UPDATE (telefono) 
ON clientes.datos_personales TO 'reception'@'localhost';

-- Ver privilegios asignados
SHOW GRANTS FOR 'app_user'@'localhost';

-- Revocar privilegios específicos
REVOKE DELETE ON tienda.* FROM 'app_user'@'localhost';
```

### **2. Políticas de Contraseñas**
```sql
-- Configurar políticas de complejidad (MySQL 8.0+)
INSTALL COMPONENT 'file://component_validate_password';
SET GLOBAL validate_password.policy = STRONG;
SET GLOBAL validate_password.length = 12;
SET GLOBAL validate_password.mixed_case_count = 2;
SET GLOBAL validate_password.number_count = 2;
SET GLOBAL validate_password.special_char_count = 1;

-- Ver políticas actuales
SELECT * FROM mysql.component WHERE component_urn LIKE '%validate_password%';
SHOW VARIABLES LIKE 'validate_password%';

-- Forzar cambio de contraseña
ALTER USER 'user1'@'localhost' PASSWORD EXPIRE;

-- Historial de contraseñas (evitar reutilización)
ALTER USER 'user1'@'localhost' 
PASSWORD HISTORY 10;
```

### **3. Autenticación y Conexiones Seguras**
```sql
-- Limitar conexiones por usuario
CREATE USER 'web_user'@'%' 
IDENTIFIED BY 'password123'
WITH MAX_QUERIES_PER_HOUR 1000
MAX_UPDATES_PER_HOUR 100
MAX_CONNECTIONS_PER_HOUR 50
MAX_USER_CONNECTIONS 10;

-- Ver conexiones actuales
SHOW PROCESSLIST;
SELECT * FROM information_schema.processlist 
WHERE USER NOT LIKE '%system%';

-- Terminar conexiones sospechosas
KILL CONNECTION 12345;

-- Configurar SSL para conexiones
CREATE USER 'secure_user'@'%'
IDENTIFIED BY 'password123'
REQUIRE SSL;

-- O forzar SSL para todos los usuarios
GRANT USAGE ON *.* TO 'app_user'@'%' REQUIRE SSL;
```

### **4. Encriptación de Datos**
```sql
-- Encriptación transparente de tablas (TDE)
CREATE TABLE datos_sensibles (
    id INT PRIMARY KEY,
    tarjeta_credito VARBINARY(255),
    ssn VARBINARY(255)
) ENCRYPTION='Y';

-- Encriptar datos manualmente
INSERT INTO datos_sensibles (id, tarjeta_credito, ssn)
VALUES (
    1,
    AES_ENCRYPT('4111111111111111', 'encryption_key'),
    AES_ENCRYPT('123-45-6789', 'encryption_key')
);

-- Desencriptar para uso legítimo
SELECT 
    id,
    CAST(AES_DECRYPT(tarjeta_credito, 'encryption_key') AS CHAR) AS tarjeta,
    CAST(AES_DECRYPT(ssn, 'encryption_key') AS CHAR) AS ssn
FROM datos_sensibles
WHERE id = 1;

-- Encriptación de conexión (configuración my.cnf)
[mysqld]
ssl-ca=/etc/mysql/ca.pem
ssl-cert=/etc/mysql/server-cert.pem
ssl-key=/etc/mysql/server-key.pem
require_secure_transport=ON
```

---

## ⚡ Optimización del Rendimiento

### **1. Configuración de Memoria (InnoDB)**
```sql
-- Ajustes clave de memoria para InnoDB
SET GLOBAL innodb_buffer_pool_size = 4294967296;  -- 4GB para servidor con 8GB RAM
SET GLOBAL innodb_buffer_pool_instances = 8;       -- Para buffer pools grandes
SET GLOBAL innodb_log_file_size = 1073741824;      -- 1GB redo logs
SET GLOBAL innodb_log_buffer_size = 67108864;      -- 64MB log buffer
SET GLOBAL innodb_flush_log_at_trx_commit = 2;     -- Balance rendimiento/durabilidad

-- Ver estadísticas del buffer pool
SELECT * FROM information_schema.INNODB_BUFFER_POOL_STATS;

-- Hit rate del buffer pool (ideal > 99%)
SELECT 
    (1 - (variable_value / (SELECT variable_value 
        FROM information_schema.global_status 
        WHERE variable_name = 'Innodb_buffer_pool_read_requests'))) * 100 AS hit_rate
FROM information_schema.global_status 
WHERE variable_name = 'Innodb_buffer_pool_reads';
```

### **2. Índices y Optimización de Consultas**
```sql
-- Analizar consultas lentas
SET GLOBAL slow_query_log = ON;
SET GLOBAL long_query_time = 2;  -- Consultas > 2 segundos
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow-queries.log';

-- Ver consultas más lentas
SELECT 
    query_time,
    lock_time,
    rows_sent,
    rows_examined,
    db,
    user_host,
    sql_text
FROM mysql.slow_log
ORDER BY query_time DESC
LIMIT 10;

-- Optimizar tablas fragmentadas
OPTIMIZE TABLE ventas, clientes, productos;

-- Analizar uso de índices
EXPLAIN ANALYZE
SELECT c.nombre, SUM(v.total) 
FROM ventas v 
JOIN clientes c ON v.cliente_id = c.id 
WHERE v.fecha BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY c.id;

-- Crear índices compuestos estratégicos
CREATE INDEX idx_ventas_fecha_cliente 
ON ventas(fecha, cliente_id, total);

CREATE INDEX idx_productos_categoria_stock 
ON productos(categoria, stock, precio);
```

### **3. Particionamiento de Tablas**
```sql
-- Particionamiento por rango (ventas históricas)
CREATE TABLE ventas_historico (
    id INT NOT NULL AUTO_INCREMENT,
    fecha DATE NOT NULL,
    cliente_id INT NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (id, fecha)
)
PARTITION BY RANGE (YEAR(fecha)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Ver datos por partición
SELECT 
    partition_name,
    table_rows,
    avg_row_length,
    data_length
FROM information_schema.partitions
WHERE table_name = 'ventas_historico';

-- Mantenimiento de particiones
-- Eliminar datos antiguos (rápido!)
ALTER TABLE ventas_historico DROP PARTITION p2020;

-- Agregar nueva partición
ALTER TABLE ventas_historico REORGANIZE PARTITION p_future INTO (
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### **4. Configuración Avanzada de Consultas**
```sql
-- Ajustar límites de MySQL
SET GLOBAL max_connections = 500;
SET GLOBAL thread_cache_size = 100;
SET GLOBAL table_open_cache = 4000;
SET GLOBAL query_cache_type = 0;  -- Desactivar en MySQL 8.0+

-- Configurar joins optimizados
SET GLOBAL optimizer_switch = 'block_nested_loop=off,batched_key_access=on';

-- Limitar consultas pesadas
SET GLOBAL max_execution_time = 30000;  -- 30 segundos máximo por consulta

-- Estadísticas de consultas
SHOW STATUS LIKE 'Com_%';
SHOW STATUS LIKE 'Innodb_%';
SHOW STATUS LIKE 'Threads_%';
```

---

## 📊 Monitoreo y Mantenimiento

### **1. Sistema de Monitoreo Integrado**
```sql
-- Performance Schema (MySQL 5.6+)
-- Activar recolección de estadísticas
UPDATE performance_schema.setup_instruments 
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE '%statement/%' OR NAME LIKE '%wait/%';

-- Consultas más costosas
SELECT 
    DIGEST_TEXT AS query,
    COUNT_STAR AS executions,
    SUM_TIMER_WAIT/1000000000 AS total_time_sec,
    AVG_TIMER_WAIT/1000000000 AS avg_time_sec,
    MAX_TIMER_WAIT/1000000000 AS max_time_sec,
    SUM_ROWS_EXAMINED AS rows_examined,
    SUM_ROWS_SENT AS rows_sent
FROM performance_schema.events_statements_summary_by_digest
WHERE DIGEST_TEXT IS NOT NULL
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Monitorear locks y deadlocks
SELECT 
    waiting_trx_id,
    waiting_pid,
    waiting_query,
    blocking_trx_id,
    blocking_pid,
    blocking_query
FROM sys.innodb_lock_waits;

-- Estadísticas de conexiones
SELECT 
    USER,
    HOST,
    DB,
    COMMAND,
    TIME,
    STATE,
    INFO
FROM information_schema.PROCESSLIST
WHERE COMMAND != 'Sleep'
ORDER BY TIME DESC;
```

### **2. Mantenimiento Programado**
```sql
-- Eventos programados para mantenimiento
DELIMITER $$
CREATE EVENT nightly_maintenance
ON SCHEDULE EVERY 1 DAY
STARTS '2024-01-01 02:00:00'
ON COMPLETION PRESERVE
ENABLE
DO
BEGIN
    -- Actualizar estadísticas
    ANALYZE TABLE ventas, clientes, productos;
    
    -- Limpiar logs antiguos
    SET @old_logs = DATE_SUB(NOW(), INTERVAL 30 DAY);
    DELETE FROM sistema.log_acceso WHERE fecha < @old_logs;
    
    -- Optimizar tablas fragmentadas
    OPTIMIZE TABLE ventas_detalle, sesiones_usuarios;
    
    -- Backup incremental de configuraciones
    INSERT INTO backup.configuraciones_backup 
    SELECT *, NOW() FROM sistema.configuraciones;
    
    -- Reporte de salud de la BD
    CALL sp_generar_reporte_salud();
END $$
DELIMITER ;

-- Ver eventos programados
SHOW EVENTS;
SELECT * FROM information_schema.EVENTS;
```

### **3. Alertas Automatizadas**
```sql
-- Procedimiento para verificar espacio en disco
DELIMITER $$
CREATE PROCEDURE sp_check_disk_space()
BEGIN
    DECLARE free_space_gb DECIMAL(10,2);
    DECLARE used_space_percent DECIMAL(5,2);
    
    -- Calcular espacio usado
    SELECT 
        ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS used_gb,
        ROUND(SUM(data_length + index_length) / disk_size * 100, 2) AS used_percent
    INTO used_space_gb, used_space_percent
    FROM information_schema.tables 
    CROSS JOIN (SELECT 100 AS disk_size) AS disk;  -- 100GB disco ejemplo
    
    -- Generar alerta si > 80%
    IF used_space_percent > 80 THEN
        INSERT INTO sistema.alertas (tipo, severidad, mensaje, fecha)
        VALUES ('DISK_SPACE', 'ALTA', 
                CONCAT('Espacio disco crítico: ', used_space_percent, '% usado'),
                NOW());
        
        -- Opcional: enviar email
        CALL sp_send_alert_email('admin@empresa.com', 
            'Alerta espacio disco MySQL',
            CONCAT('MySQL usa ', used_space_percent, '% del disco.'));
    END IF;
END $$
DELIMITER ;
```

---

## 💾 Backup y Recovery

### **1. Estrategias de Backup**
```sql
-- Backup lógico completo (mysqldump desde consola)
# mysqldump -u root -p --single-transaction --routines --triggers 
# --events --databases tienda clientes productos > backup_completo.sql

-- Backup incremental con binlogs
# mysqlbinlog --start-datetime="2024-01-01 00:00:00" 
# mysql-bin.000001 mysql-bin.000002 > backup_incremental.sql

-- Script de backup automatizado
DELIMITER $$
CREATE PROCEDURE sp_backup_diario()
BEGIN
    SET @fecha = DATE_FORMAT(NOW(), '%Y%m%d_%H%i%s');
    SET @archivo = CONCAT('/backups/mysql/full_', @fecha, '.sql');
    
    -- Ejecutar mysqldump via sistema
    SET @comando = CONCAT(
        'mysqldump -u root -pPASSWORD ',
        '--single-transaction --routines --triggers --events ',
        '--all-databases > ', @archivo
    );
    
    -- Registrar en log
    INSERT INTO sistema.backup_log (tipo, archivo, tamaño, fecha_inicio)
    VALUES ('FULL', @archivo, 0, NOW());
    
    -- En producción, usar eventos del sistema o cron jobs
END $$
DELIMITER ;
```

### **2. Point-in-Time Recovery**
```sql
-- 1. Restaurar backup completo
# mysql -u root -p < backup_completo.sql

-- 2. Aplicar binlogs hasta punto específico
# mysqlbinlog --stop-datetime="2024-01-15 14:30:00" 
# mysql-bin.000003 mysql-bin.000004 | mysql -u root -p

-- Procedimiento para recovery testing
DELIMITER $$
CREATE PROCEDURE sp_test_recovery()
BEGIN
    DECLARE exit_code INT;
    
    -- Crear BD de prueba
    CREATE DATABASE IF NOT EXISTS recovery_test;
    
    -- Restaurar backup en BD de prueba
    SET @restore_cmd = CONCAT(
        'mysql -u root -pPASSWORD recovery_test < ',
        '/backups/mysql/latest_backup.sql'
    );
    
    -- Verificar integridad
    CALL sp_verificar_integridad('recovery_test');
    
    -- Log resultado
    INSERT INTO sistema.recovery_tests (fecha, resultado, observaciones)
    VALUES (NOW(), 'EXITOSO', 'Recovery test completado');
    
    -- Limpiar
    DROP DATABASE recovery_test;
END $$
DELIMITER ;
```

### **3. Replicación para Alta Disponibilidad**
```sql
-- Configurar maestro (master)
-- En my.cnf del maestro:
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
binlog-row-image = FULL

-- Crear usuario replicación
CREATE USER 'replicador'@'%' IDENTIFIED BY 'Rep1ic@t10nP@ss';
GRANT REPLICATION SLAVE ON *.* TO 'replicador'@'%';

-- Ver estado del maestro
SHOW MASTER STATUS;

-- Configurar esclavo (slave)
-- En my.cnf del esclavo:
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
read-only = 1

-- Conectar esclavo a maestro
CHANGE MASTER TO
MASTER_HOST = 'master_ip',
MASTER_USER = 'replicador',
MASTER_PASSWORD = 'Rep1ic@t10nP@ss',
MASTER_LOG_FILE = 'mysql-bin.000001',
MASTER_LOG_POS = 107;

-- Iniciar replicación
START SLAVE;

-- Ver estado replicación
SHOW SLAVE STATUS\G
```

---

## 🛠️ Herramientas de Administración

### **Comandos Esenciales del DBA**
```bash
# Monitoreo en tiempo real
mysqladmin -u root -p processlist
mysqladmin -u root -p status
mytop --user=root --password

# Analizar logs
tail -f /var/log/mysql/error.log
mysqldumpslow /var/log/mysql/slow-queries.log

# Optimización
mysqlcheck -u root -p --analyze --all-databases
mysqlcheck -u root -p --optimize --all-databases

# Seguridad
mysql_secure_installation
mysql_upgrade -u root -p

# Backup físico (InnoDB)
# Requiere detener MySQL o usar LVM snapshots
```

### **Script de Health Check Automático**
```sql
DELIMITER $$
CREATE PROCEDURE sp_health_check()
BEGIN
    DECLARE health_score INT DEFAULT 100;
    DECLARE warnings INT DEFAULT 0;
    DECLARE errors INT DEFAULT 0;
    DECLARE message TEXT;
    
    -- Verificar conexiones
    IF (SELECT COUNT(*) FROM information_schema.PROCESSLIST) > 300 THEN
        SET health_score = health_score - 10;
        SET warnings = warnings + 1;
        SET message = CONCAT_WS(', ', message, 'Muchas conexiones activas');
    END IF;
    
    -- Verificar locks
    IF EXISTS (SELECT 1 FROM sys.innodb_lock_waits) THEN
        SET health_score = health_score - 20;
        SET warnings = warnings + 1;
        SET message = CONCAT_WS(', ', message, 'Deadlocks detectados');
    END IF;
    
    -- Verificar replicación
    IF EXISTS (SELECT 1 
               FROM information_schema.PROCESSLIST 
               WHERE COMMAND = 'Binlog Dump') THEN
        -- Replicación activa, verificar delay
        IF (SELECT Slave_lag FROM sys.replication_status) > 60 THEN
            SET health_score = health_score - 15;
            SET warnings = warnings + 1;
            SET message = CONCAT_WS(', ', message, 'Replicación con retraso');
        END IF;
    END IF;
    
    -- Retornar resultado
    SELECT 
        health_score AS health_score,
        warnings,
        errors,
        message AS issues,
        NOW() AS check_time;
END $$
DELIMITER ;
```

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Perfil_Profesional-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)
[![GitHub](https://img.shields.io/badge/GitHub-Repositorios-black?style=flat&logo=github)](https://github.com/dovalless)

💼 **Administrador de Bases de Datos**  
🎓 **Certificado en Administración MySQL por Alura**  
🛡️ **Especialista en Seguridad y Optimización de BD**

*"La administración profesional de bases de datos no es un lujo, es una necesidad. En un mundo donde los datos son el activo más valioso, saber protegerlos, optimizar su acceso y garantizar su disponibilidad es una de las habilidades más críticas para cualquier organización. Este curso proporciona las herramientas esenciales para convertirte en un DBA eficiente y proactivo."*

**#AluraLatam #MySQLAdmin #DatabaseSecurity #PerformanceTuning #DBA**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Alura Latam** - Por formación técnica de alto nivel
- **Comunidad MySQL** - Por documentación y herramientas open source
- **Instructores del curso** - Por compartir experiencia práctica
- **Profesionales DBA** - Por establecer estándares de excelencia

<div align="center">

### ⭐ Un DBA preparado previene crisis, no solo las resuelve ⭐

### 🚀 ¡Administra con seguridad, escala con confianza! 🚀

**Conocimientos aplicados con 🛡️ y ⚡ por Darwin Ovalles**

---
*Buenas prácticas de administración | MySQL 8.0+ | Entornos de producción*

</div>
