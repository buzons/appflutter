## main
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


#homepage

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



# buscar

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
                    '\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}', // Precio formateado
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


#detalle productos

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


# error page

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



# datos page

import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:sena_project/widgets/producto_details.dart';
import 'package:sena_project/widgets/search_page.dart';
import '../models/product.dart';
import '../widgets/carrito_compra.dart'; // Importa la página del carrito

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
  List<Products> _cartItems = []; // Lista para almacenar los productos en el carrito
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
            return Container(); // Devuelve un contenedor vacío si no coincide con la búsqueda
          }
          return Card(
            margin: EdgeInsets.all(10.0),
            child: InkWell(
              onTap: () {
                ProductDetailsDialog.show(context, product); // Muestra los detalles del producto
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
                      '\$${NumberFormat('#,##0.00', 'en_US').format(product.price)}', // Precio formateado
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
                _cartItems = value ?? []; // Si el valor es null, se asigna una lista vacía
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


# carrito compra

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
                _removeFromCart(product); // Elimina el producto del carrito
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
                  _proceedToCheckout(); // Implementa la lógica para proceder con el pago o agregar más productos
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
    // Implementa la lógica para proceder con el pago o agregar más productos
  }
}


#carga page

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


