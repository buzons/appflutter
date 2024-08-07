### main
```
import 'package:flutter/material.dart';
import 'home_page.dart';

void main(){
  runApp(Miclase());
}

class Miclase extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
   return MaterialApp(
     title: 'Farmacia Constantine',
     debugShowCheckedModeBanner: false,
    home: HomePage(),
   );
 } 
}
```

### homepage
```
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'package:sena_project/models/product.dart';
import 'widgets/carga_page.dart';
import 'widgets/datos_page.dart';
import 'widgets/error_page.dart';

class HomePage extends StatelessWidget {
  const HomePage({
    Key? key,
  }) : super(key: key);

 @override
  Widget build(BuildContext context) {
    return Scaffold(
     
      body: FutureBuilder(
        builder: (BuildContext context, AsyncSnapshot<List<Products>> snapshot) {
          if (snapshot.connectionState == ConnectionState.done) {
            if (snapshot.hasError) {
              print('Error: ${snapshot.data}');
              print('Error: ${snapshot.error}');
              return ErrorPage();
              
            }else if (snapshot.hasData) {
              return DatoPage(products: snapshot.data as List<Products>);
            }
          }
          return CargaPage();
        },
        future: getList(),
      ),
    );
  }


  Future<List<Products>> getList() async {
    final url = Uri.parse('https://76b7-181-78-4-250.ngrok-free.app/products');
    final response = await http.get(url);
    if (response.statusCode == 200){
      final info = jsonDecode(response.body);
      print(info);
      return productsFromJson(jsonEncode(info));
    }else{
      throw 'Error';
    }
  }
}

```

### buscar
```
import 'package:flutter/material.dart';
import 'package:sena_project/models/product.dart';
import 'package:intl/intl.dart';
import 'package:sena_project/widgets/producto_details.dart';


class ProductSearch extends SearchDelegate<Products> {
  final List<Products> products;
  
  ProductSearch(this.products);

  @override
  List<Widget> buildActions(BuildContext context) {
    return [
      IconButton(
        icon: Icon(Icons.clear),
        onPressed: () {
          query = '';
        },
      ),
    ];
  }

  @override
  Widget buildLeading(BuildContext context) {
    return IconButton(
      icon: Icon(Icons.arrow_back),
      onPressed: () {
        Navigator.pop(context);
      },
    );
  }

  @override
  Widget buildResults(BuildContext context) {
    final List<Products> results = products
        .where((product) => product.name.toLowerCase().contains(query.toLowerCase()))
        .toList();

    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 10.0,
        mainAxisSpacing: 10.0,
        childAspectRatio: 0.7,
      ),
      itemCount: results.length,
      itemBuilder: (context, index) {
        final product = results[index];
        return Card(
          margin: EdgeInsets.all(10.0),
          child: InkWell(
            onTap: () {
                ProductDetailsDialog.show(context, product);
            },
            child: Padding(
              padding: EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: <Widget>[
                  Container(
                    width: 80,
                    height: 80,
                    decoration: BoxDecoration(
                      shape: BoxShape.rectangle,
                      image: DecorationImage(
                        fit: BoxFit.cover,
                        image: NetworkImage(product.icon),
                      ),
                    ),
                  ),
                  Text(
                    product.name,
                    style: TextStyle(
                      fontSize: 20.0,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 8.0),
                  Text(
                    product.description,
                    style: TextStyle(fontSize: 16.0),
                  ),
                  SizedBox(height: 8.0),
                  Text(
                    '\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}', 
                    style: TextStyle(
                      fontSize: 18.0,
                      fontWeight: FontWeight.bold,
                      color: Colors.green,
                    ),
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }

  @override
  Widget buildSuggestions(BuildContext context) {
    final List<Products> suggestions = products
        .where((product) => product.name.toLowerCase().contains(query.toLowerCase()))
        .toList();

    return ListView.builder(
      itemCount: suggestions.length,
      itemBuilder: (context, index) {
        final product = suggestions[index];
        return ListTile(
          title: Text(product.name),
          onTap: () {
            query = product.name;
            showResults(context);
          },
        );
      },
    );
  }
 }

```

### detalle productos
```
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../models/product.dart';

class ProductDetailsDialog {
  static void show(BuildContext context, Products product) {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: Text(product.name),
          content: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            mainAxisSize: MainAxisSize.min,
            children: <Widget>[
              Image.network(
                product.icon,
                width: 150,
                height: 150,
                fit: BoxFit.cover,
              ),
              SizedBox(height: 16.0),
              Text(
                product.description,
                style: TextStyle(fontSize: 16.0),
              ),
              SizedBox(height: 8.0),
              Text(
                '\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}',
                style: TextStyle(
                  fontSize: 18.0,
                  fontWeight: FontWeight.bold,
                  color: Colors.green,
                ),
              ),
            ],
          ),
          actions: <Widget>[
            TextButton(
              child: Text('Cerrar'),
              style: TextButton.styleFrom(
                foregroundColor: Color.fromARGB(255, 115, 76, 122),
              ),
              onPressed: () {
                Navigator.of(context).pop();
              },
            ),
            TextButton(
              child: Text('Comprar'),
              style: TextButton.styleFrom(
                foregroundColor: Color.fromARGB(255, 115, 76, 122),
              ),
              onPressed: () {
               
               
              },
            ),
          ],
        );
      },
    );
  }
}

```

### error page
```
import 'package:flutter/material.dart';

class ErrorPage extends StatelessWidget {
  const ErrorPage({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Center(child: Text('ERROR 404'));
  }
}
```


### datos page
```
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:sena_project/widgets/producto_details.dart';
import 'package:sena_project/widgets/search_page.dart';
import '../models/product.dart';
import '../widgets/carrito_compra.dart';

class DatoPage extends StatefulWidget {
  final List<Products> products;

  const DatoPage({
    Key? key,
    required this.products,
  }) : super(key: key);

  @override
  _DatoPageState createState() => _DatoPageState();
}

class _DatoPageState extends State<DatoPage> {
  List<Products> _cartItems = []; 
  String _searchQuery = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(
          'Farmacia Constantine',
          style: TextStyle(
            color: Colors.white,
          ),
        ),
        backgroundColor: Color.fromARGB(255, 115, 76, 122), // Color de fondo del app bar
        elevation: 0, // Sin sombra bajo el app bar
        actions: [
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12.0),
            child: IconButton(
              icon: Icon(Icons.search),
              color: Colors.white,
              onPressed: () {
                showSearch(context: context, delegate: ProductSearch(widget.products));
              },
            ),
          ),
        ],
      ),
      body: GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 10.0,
          mainAxisSpacing: 10.0,
          childAspectRatio: 0.7,
        ),
        itemCount: widget.products.length,
        itemBuilder: (context, index) {
          final product = widget.products[index];
          // Aplica la lógica de filtro según el texto de búsqueda
          if (_searchQuery.isNotEmpty &&
              !product.name.toLowerCase().contains(_searchQuery.toLowerCase())) {
            return Container(); 
          }
          return Card(
            margin: EdgeInsets.all(10.0),
            child: InkWell(
              onTap: () {
                ProductDetailsDialog.show(context, product);
              },
              child: Padding(
                padding: EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: <Widget>[
                    Container(
                      width: 80,
                      height: 80,
                      decoration: BoxDecoration(
                        shape: BoxShape.rectangle,
                        image: DecorationImage(
                          fit: BoxFit.cover,
                          image: NetworkImage(product.icon),
                        ),
                      ),
                    ),
                    Text(
                      product.name,
                      style: TextStyle(
                        fontSize: 20.0,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    SizedBox(height: 8.0),
                    Text(
                      product.description,
                      style: TextStyle(fontSize: 16.0),
                    ),
                    SizedBox(height: 8.0),
                    Text(
                      '\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}', 
                      style: TextStyle(
                        fontSize: 18.0,
                        fontWeight: FontWeight.bold,
                        color: Colors.green,
                      ),
                    ),
                    SizedBox(height: 8.0),
                    ElevatedButton(
                      onPressed: () {
                        _addToCart(product);
                      },
                      
                      child: Text('Agregar al carrito'),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () {
          if (_cartItems.isNotEmpty) {
            Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => CarritoPage(cartItems: _cartItems),
              ),
            ).then((value) {
              // Actualiza la lista _cartItems con el valor devuelto
              setState(() {
                _cartItems = value ?? []; 
              });
            });
          } else {
            showDialog(
              context: context,
              builder: (context) => AlertDialog(
                title: Text('Carrito de compras vacío'),
                content: Text('No hay productos en el carrito para proceder con el pago.'),
                actions: [
                  TextButton(
                    onPressed: () {
                      Navigator.of(context).pop();
                    },
                    child: Text('Aceptar'),
                  ),
                ],
              ),
            );
          }
        },
        icon: Icon(Icons.shopping_cart),
        label: Text('${_cartItems.length}'), // Muestra la cantidad total de productos en el carrito
        backgroundColor: Color.fromARGB(255, 115, 76, 122),
        foregroundColor: Colors.white,
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,
    );
  }

  void _addToCart(Products product) {
    setState(() {
      _cartItems.add(product);
    });
  }
}
```

### carrito compra
```
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../models/product.dart';

class CarritoPage extends StatefulWidget {
  final List<Products> cartItems;

  const CarritoPage({Key? key, required this.cartItems}) : super(key: key);

  @override
  _CarritoPageState createState() => _CarritoPageState();
}

class _CarritoPageState extends State<CarritoPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Carrito de compras'),
      ),
      body: ListView.builder(
        itemCount: widget.cartItems.length,
        itemBuilder: (context, index) {
          final product = widget.cartItems[index];
          return ListTile(
            leading: CircleAvatar(
              backgroundImage: NetworkImage(product.icon),
            ),
            title: Text(product.name),
            subtitle: Text('\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}'),
            trailing: IconButton(
              icon: Icon(Icons.delete),
              onPressed: () {
                _removeFromCart(product); 
              },
            ),
          );
        },
      ),
      bottomNavigationBar: BottomAppBar(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: <Widget>[
              Text(
                'Subtotal: \$${_calculateSubtotal()}',
                style: TextStyle(fontSize: 18.0, fontWeight: FontWeight.bold),
              ),
              ElevatedButton(
                onPressed: () {
                  _proceedToCheckout(); 
                },
                child: Text('Pagar'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _removeFromCart(Products product) {
    setState(() {
      widget.cartItems.remove(product);
    });
  }

  String _calculateSubtotal() {
    double subtotal = 0.0;
    for (var item in widget.cartItems) {
      subtotal += item.price;
    }
    return NumberFormat('#,##0.00', 'en_US').format(subtotal);
  }

  void _proceedToCheckout() {
    
  }
}
```

## carga page
```
import 'package:flutter/material.dart';

class CargaPage extends StatelessWidget {
  const CargaPage({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Center(child: CircularProgressIndicator());
  }
}
```
## login page
```
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:sena_project/models/product.dart';
import 'package:sena_project/widgets/datos_page.dart'; // Asegúrate de que esta ruta sea correcta

class LoginPage extends StatefulWidget {
  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool _isLoading = false;
  String? _errorMessage;

  Future<void> _login() async {
    setState(() {
      _isLoading = true;
      _errorMessage = null;
    });

    final url = Uri.parse("https://275c-181-78-4-250.ngrok-free.app/users"); // Reemplaza con tu endpoint de usuarios
    try {
      final response = await http.get(url);

      setState(() {
        _isLoading = false;
      });

      if (response.statusCode == 200) {
        final List<dynamic> data = jsonDecode(response.body);

        // Busca el usuario en la lista obtenida
        final userExists = data.any(
          (user) =>
              user['email'] == _usernameController.text &&
              user['password'] == _passwordController.text,
        );

        if (userExists) {
          // Obtener la lista de productos después de validar al usuario
          await _fetchProductsAndNavigate();
        } else {
          setState(() {
            _errorMessage = 'Email o contraseña incorrectos';
          });
        }
      } else {
        setState(() {
          _errorMessage = 'Error en el servidor. Inténtalo nuevamente.';
        });
      }
    } catch (e) {
      setState(() {
        _isLoading = false;
        _errorMessage = 'Ocurrió un error. Revisa tu conexión y vuelve a intentar.';
      });
      print("Error de conexión: $e");
    }
  }

  Future<void> _fetchProductsAndNavigate() async {
    setState(() {
      _isLoading = true;
    });

    final url = Uri.parse("https://275c-181-78-4-250.ngrok-free.app/products"); // Reemplaza con tu endpoint de productos
    try {
      final response = await http.get(url);

      setState(() {
        _isLoading = false;
      });

      if (response.statusCode == 200) {
        final List<dynamic> productData = jsonDecode(response.body);

        // Convierte los productos a una lista de objetos Products
        List<Products> products = productsFromJson(jsonEncode(productData));

        // Navega a DatoPage con la lista de productos
        Navigator.pushReplacement(
          context,
          MaterialPageRoute(
            builder: (context) => DatoPage(products: products),
          ),
        );
      } else {
        setState(() {
          _errorMessage = 'Error al obtener productos. Inténtalo nuevamente.';
        });
      }
    } catch (e) {
      setState(() {
        _isLoading = false;
        _errorMessage = 'Ocurrió un error al obtener productos. Revisa tu conexión y vuelve a intentar.';
      });
      print("Error de conexión: $e");
    }
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: const BoxDecoration(
          gradient: LinearGradient(
        begin: Alignment.topRight,
        end: Alignment.bottomLeft,
        colors: [
          Colors.purple, // Color de fondo morado
          Colors.deepPurple, // Color de fondo morado oscuro
        ],
      )),
      child: Scaffold(
        backgroundColor: Colors.transparent,
        body: Padding(
          padding: const EdgeInsets.all(32.0),
          child: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                
                const SizedBox(height: 50),
                _inputField("Email", _usernameController),
                const SizedBox(height: 20),
                _inputField("Password", _passwordController, isPassword: true),
                const SizedBox(height: 50),
                _isLoading
                    ? CircularProgressIndicator()
                    : _loginBtn(),
                const SizedBox(height: 20),
                if (_errorMessage != null)
                  Text(
                    _errorMessage!,
                    style: TextStyle(color: Colors.red),
                  ),
                _extraText(),
              ],
            ),
          ),
        ),
      ),
    );
  }

  

  Widget _inputField(String hintText, TextEditingController controller,
      {isPassword = false}) {
    var border = OutlineInputBorder(
        borderRadius: BorderRadius.circular(18),
        borderSide: const BorderSide(color: Colors.white));

    return TextField(
      style: const TextStyle(color: Colors.white),
      controller: controller,
      decoration: InputDecoration(
        hintText: hintText,
        hintStyle: const TextStyle(color: Colors.white),
        enabledBorder: border,
        focusedBorder: border,
      ),
      obscureText: isPassword,
    );
  }

  Widget _loginBtn() {
    return ElevatedButton(
      onPressed: () {
        _login();
      },
      child: const SizedBox(
          width: double.infinity,
          child: Text(
            "Sign in",
            textAlign: TextAlign.center,
            style: TextStyle(fontSize: 20),
          )),
      style: ElevatedButton.styleFrom(
        foregroundColor: Colors.purple, // Color del texto del botón
        shape: const StadiumBorder(), 
        backgroundColor: Colors.white, // Color de fondo del botón
        padding: const EdgeInsets.symmetric(vertical: 16),
      ),
    );
  }

  Widget _extraText() {
    return const Text(
      "Can't access to your account?",
      textAlign: TextAlign.center,
      style: TextStyle(fontSize: 16, color: Colors.white),
    );
  }
}
```

## main (login_page)
```
import 'package:flutter/material.dart';

import 'package:sena_project/widgets/login2.dart';


void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Login',
      debugShowCheckedModeBanner: false,
      
      home: LoginPage(),
    );
  }
}

```
