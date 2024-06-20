# main 
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
