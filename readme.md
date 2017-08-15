# {{db}}  - Gopher Sauce extension

Install database pipelines to your templates. Perform Mongo database actions with Golang [templates](https://golang.org/pkg/text/template/).

1. [Guide](#how-it-works)
2. [pipelines 🐧](#pipelines)

Add this `import` tag within your `.gxml` file.

	<import src="github.com/cheikhshift/db/gos.gxml"/>

#How it works

## Save object

1. Declare a struct within your `header` tag in your `GoS` file. This struct will be named `Sample`. It is a must to have the `Id` field.

		<struct name="Sample">
	 		Id bson.ObjectId `bson:"_id,omitempty"`
			TestField string `valid:"unique"`
			FieldTwo string `valid:"email,unique,required"`
			Created time.Time //timestamp local format
			Updated time.Time			
		</struct>

2. Create a new file named `index.tmpl` in your `tmpl` directory. Contents of file :

		<!DOCTYPE html>
			<html lang="en">
		    <head>
		        <title>DB Example</title>
		    </head>
		    <body>
				<!-- pipelines will go here -->
		    </body>
		</html>

3. Add these following lines in `<!-- pipelines will go here -->` : 
4. to create a new `Sample` within your `index.tmpl`. This will auto-populate `Id` and `Created`.
 
			{{ $obj := New Sample }}

4. Update values of `$obj` to prevent validation errors

		{{with $obj }}
           	{{ .|u "FieldTwo" "email@gmail.com" }}
        {{end}}

5. Save `$obj` to database :

		{{ $err := $obj | Update }} 

6. Check if error occurred and display :

	        {{ if $err | neq "" }}
         		<h1>Error {{$err}}</h1>
	        {{end}}


## Get object
1. Add these following lines in `<!-- pipelines will go here -->` : 
2. Load a new query map :
	
		  {{ $query := search }}

3. Set query keys :
	
	        {{with $query }}
	        	{{ . | k "fieldtwo" "email@gmail.com" }}
	        {{end}}

4. Perform query and display one item :

	     	{{ with $query | Q Sample}}
	            {{ . | One }}
            {{ end }}

5. Perform a query and attempt update. The pipeline `isSample` will convert the JSON database object to a `Sample` struct reader.
	

		{{ with $query | Q Sample}}
            {{ range . | All }}
            
                    {{ . | k "fieldtwo" "newmail" }}
                    <!-- Cast similar to with $obj -->
                    {{ with . | isSample }}
                       FieldTwo : {{ .FieldTwo }}
                       <!-- will display invalid email -->
                       {{ . | Update }}
                    {{end }}
            {{ end }}
        {{ end }}

#Pipelines

### `{{  New param }}`
Returns Struct with `Id` and `Created` initialized. 

- param : `Struct` name without strings or any braces. For example a struct declared in your `.gxml` file named `Sample` will be initialized with pipeline ` {{ $obj := New Sample }}`

### `{{  $param | Update   }}`
If an error occurs while updating your object this pipeline will return a string error as to why.

- $param :  Struct  to save to database.

### `{{ $param | Delete }}`
If an error occurs while updating your object this pipeline will return a string error as to why.

- $param : Struct to remove from database. Requires a valid ObjectId in field `Id` to work.

### `{{ $param | u "$param1" $param2  }}`
Update a field value within your struct.

- $param : Struct to update.
- $param1 : Struct field name to update.
- $param2 : New value to set to specified struct field.

### `{{ search }}`
Returns a new query map (= json ).

### `{{ $search | k "$param1" $param2 }}`
Set a key within your query map. This is the equivalent of updating a map value within `Go`. 

- $search : Query map.
- $param1 : String of key to set.
- $param2 : Value to set.

### `{{ $search | rmk "$param1" }}`
Remove key from query map.

- $search : Query map.
- $param1 : String of key to remove.

### `{{ $search | v "$param1" }}`
Return value of specified query map key.

- $search : Query map.
- $param1 : String of key to retrieve.

### `{{ $search | Q $param1 }}`
Return new `*mgo.Query`.

- $search : Query map.
- $param1 : `Struct` name without strings or any braces. For example a struct declared in your `.gxml` file named `Sample` will be initialized with pipeline ` {{ $search | Q Sample }}`

### `{{ Limit $param $param1   }}`
Limit query to specified number.

- $param : *mgo.Query to limit.
- $param1 : int value to limit query by.

### `{{ Skip $param $param1 }}`
Skip query by specified number.

- $param : *mgo.Query to limit.
- $param1 : int value to skip query by.

### `{{ Sort $param $param1 }}`
Sort query by specified field. Any struct field must be in lower case.

- $param : *mgo.Query to limit.
- $param1 :  string of field and sort syntax. For example `{{ Sort $query "-firstname" }}`

### `{{  $param | Count  }}`
Return int length of query.

- $param : *mgo.Query to count.

### `{{ $param | All }}`
Return `slice` of  maps of your query. 

- $param : *mgo.Query to count.

### `{{ $param | One }}`
Return one result of your query as a map.

- $param : *mgo.Query to find one of.

## Cast trick
Use the following syntax to convert a map to a struct for item update or removal. Only structs declared within your project's `gxml` files are available to your templates

	 {{  $map | is<GoS Struct name> }}