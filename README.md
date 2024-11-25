Got it! Let’s pick up from the beginning, assuming the tables are already created. I’ll guide you through the process of populating the data, creating the stored procedure, and testing it. Here’s everything laid out step-by-step.

Step 1: Populate Tables with Sample Data

Populate the tables with relevant data to ensure we can test permissions functionality effectively.

1.1 Insert Data into the Permission Table

This table defines the types of permissions:

INSERT INTO Permission (ID_Permission, Nombre, Descripcion)
VALUES 
(1, 'Lectura', 'Permite leer registros'), -- Read Permission
(2, 'Escritura', 'Permite modificar registros'), -- Write Permission
(3, 'Eliminación', 'Permite borrar registros'); -- Delete Permission

1.2 Insert Data into the PermiUser Table

This table assigns permissions to users for entire entities:

INSERT INTO PermiUser (ID_User, ID_Entity, ID_Permission)
VALUES 
(1, 101, 1), -- User 1 has Read access on Entity 101
(1, 102, 2); -- User 1 has Write access on Entity 102

1.3 Insert Data into the PermiUserRecord Table

This table assigns permissions to users for specific records within entities:

INSERT INTO PermiUserRecord (ID_User, ID_Entity, ID_Record, ID_Permission)
VALUES 
(1, 101, 1001, 2), -- User 1 has Write access on Record 1001 in Entity 101
(1, 101, 1002, 1); -- User 1 has Read access on Record 1002 in Entity 101

1.4 Insert Data into the PermiRole Table

This table assigns permissions to roles for entire entities:

INSERT INTO PermiRole (ID_Role, ID_Entity, ID_Permission)
VALUES 
(1, 101, 3); -- Role 1 has Delete access on Entity 101

1.5 Insert Data into the PermiRoleRecord Table

This table assigns permissions to roles for specific records:

INSERT INTO PermiRoleRecord (ID_Role, ID_Entity, ID_Record, ID_Permission)
VALUES 
(1, 101, 1001, 1), -- Role 1 has Read access on Record 1001 in Entity 101
(1, 102, 2001, 2); -- Role 1 has Write access on Record 2001 in Entity 102

Step 2: Create the Stored Procedure

The procedure will take a user ID and an entity ID and return permissions assigned to the user at both the entity and record levels.

CREATE PROCEDURE GetUserPermissions
    @ID_User INT,
    @ID_Entity INT
AS
BEGIN
    -- Permissions assigned directly to the user for the entity
    SELECT 
        'User' AS PermissionType,
        u.ID_User,
        u.ID_Entity,
        NULL AS ID_Record,
        u.ID_Permission,
        p.Nombre AS PermissionName
    FROM 
        PermiUser u
    INNER JOIN 
        Permission p ON u.ID_Permission = p.ID_Permission
    WHERE 
        u.ID_User = @ID_User AND u.ID_Entity = @ID_Entity

    UNION ALL

    -- Permissions assigned to the user for specific records
    SELECT 
        'UserRecord' AS PermissionType,
        ur.ID_User,
        ur.ID_Entity,
        ur.ID_Record,
        ur.ID_Permission,
        p.Nombre AS PermissionName
    FROM 
        PermiUserRecord ur
    INNER JOIN 
        Permission p ON ur.ID_Permission = p.ID_Permission
    WHERE 
        ur.ID_User = @ID_User AND ur.ID_Entity = @ID_Entity

    UNION ALL

    -- Permissions assigned to the user's role for the entity
    SELECT 
        'Role' AS PermissionType,
        NULL AS ID_User,
        r.ID_Entity,
        NULL AS ID_Record,
        r.ID_Permission,
        p.Nombre AS PermissionName
    FROM 
        PermiRole r
    INNER JOIN 
        Permission p ON r.ID_Permission = p.ID_Permission
    INNER JOIN 
        UserRoles ur ON ur.ID_Role = r.ID_Role -- Assuming a UserRoles table exists
    WHERE 
        ur.ID_User = @ID_User AND r.ID_Entity = @ID_Entity

    UNION ALL

    -- Permissions assigned to the user's role for specific records
    SELECT 
        'RoleRecord' AS PermissionType,
        NULL AS ID_User,
        rr.ID_Entity,
        rr.ID_Record,
        rr.ID_Permission,
        p.Nombre AS PermissionName
    FROM 
        PermiRoleRecord rr
    INNER JOIN 
        Permission p ON rr.ID_Permission = p.ID_Permission
    INNER JOIN 
        UserRoles ur ON ur.ID_Role = rr.ID_Role -- Assuming a UserRoles table exists
    WHERE 
        ur.ID_User = @ID_User AND rr.ID_Entity = @ID_Entity;
END;

Step 3: Test the Stored Procedure

3.1 Test Case: User with Entity-Level Permissions

Call the procedure for a user with entity-level permissions:

EXEC GetUserPermissions @ID_User = 1, @ID_Entity = 101;

Expected Output

PermissionType	ID_User	ID_Entity	ID_Record	ID_Permission	PermissionName
User	1	101	NULL	1	Lectura
UserRecord	1	101	1001	2	Escritura
UserRecord	1	101	1002	1	Lectura
Role	NULL	101	NULL	3	Eliminación
RoleRecord	NULL	101	1001	1	Lectura

3.2 Test Case: User with No Permissions

Call the procedure for a user without permissions:

EXEC GetUserPermissions @ID_User = 2, @ID_Entity = 101;

Expected Output

No rows should be returned.

3.3 Test Case: User with Role-Based Permissions

Call the procedure for a user whose permissions come from their role:

EXEC GetUserPermissions @ID_User = 1, @ID_Entity = 102;

Expected Output

PermissionType	ID_User	ID_Entity	ID_Record	ID_Permission	PermissionName
User	1	102	NULL	2	Escritura
RoleRecord	NULL	102	2001	2	Escritura

Step 4: Debugging

	1.	If no data is returned, ensure the UserRoles table correctly maps users to roles.
	2.	Check for typos in column names or table references.
	3.	Ensure joins are correct (e.g., UserRoles.ID_Role matches PermiRole.ID_Role).

Let me know if further clarification is needed!
