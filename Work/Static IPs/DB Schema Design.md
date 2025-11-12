
```sql

CREATE TABLE `user_registered_ips` (
  `id`              INT PRIMARY KEY,
  `created_at`      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at`      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `version`         INT NOT NULL DEFAULT 1,
  `user_account_id` VARCHAR(36),
  `ip_address`      VARCHAR(45) NOT NULL,
  `is_primary`      TINYINT(1) NOT NULL DEFAULT 1,
  `is_active`       TINYINT(1) NOT NULL DEFAULT 1,
   
   UNIQUE KEY ux_user_ip_active (user_account_id, ip_address, is_active),
   UNIQUE KEY ux_one_primary_active (user_account_id, is_primary, is_active) 
)
```