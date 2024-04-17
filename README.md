### the code of Boxify 
simple note app coded on vue and directus (database), designed with vuetify 

database structure (you need this to open up the project): 

TABLE: box 
  
  fields: 
  - id,
  - boxTitle (String),
  - boxType (int),
  - isEditing (int)
    
TABLE: item

  fields: 
  - id,
  - itemName (String),
  - boxId (int),
  - isCharacterize (int),
  - col (int)

local database on port 8055

check out new version boxify 2.0 with cleaner html code and designed in tailwind
