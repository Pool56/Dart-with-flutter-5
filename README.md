# Dart-with-flutter-5
# Create a new Flutter project
flutter create movie_analysis
cd movie_analysis
# Addition of dependencies
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.3
  provider: ^6.0.0
  # Designing if the UI
  import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'movie_provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => MovieProvider(),
      child: MaterialApp(
        title: 'Movie Analysis',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: MovieScreen(),
      ),
    );
  }
}

class MovieScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final movieProvider = Provider.of<MovieProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Movie Analysis'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              decoration: InputDecoration(labelText: 'Enter Movie Title'),
              onSubmitted: (value) {
                movieProvider.fetchMovie(value);
              },
            ),
            SizedBox(height: 20),
            movieProvider.isLoading
                ? CircularProgressIndicator()
                : movieProvider.movie == null
                    ? Container()
                    : Expanded(
                        child: ListView(
                          children: [
                            Text(
                              movieProvider.movie!.title,
                              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                            ),
                            Text('Release Date: ${movieProvider.movie!.releaseDate}'),
                            Text('Overview: ${movieProvider.movie!.overview}'),
                          ],
                        ),
                      ),
          ],
        ),
      ),
    );
  }
}
# Creation of  the provider and fetching movie data
import 'package:flutter/material.dart';
import 'dart:convert';
import 'package:http/http.dart' as http;

class Movie {
  final String title;
  final String releaseDate;
  final String overview;

  Movie({
    required this.title,
    required this.releaseDate,
    required this.overview,
  });

  factory Movie.fromJson(Map<String, dynamic> json) {
    return Movie(
      title: json['title'],
      releaseDate: json['release_date'],
      overview: json['overview'],
    );
  }
}

class MovieProvider with ChangeNotifier {
  Movie? _movie;
  bool _isLoading = false;

  Movie? get movie => _movie;
  bool get isLoading => _isLoading;

  Future<void> fetchMovie(String title) async {
    _isLoading = true;
    notifyListeners();

    final response = await http.get(Uri.parse(
        'https://api.themoviedb.org/3/search/movie?api_key=YOUR_API_KEY&query=$title'));

    if (response.statusCode == 200) {
      final jsonResponse = json.decode(response.body);
      final movieJson = jsonResponse['results'][0];
      _movie = Movie.fromJson(movieJson);
    } else {
      throw Exception('Failed to load movie');
    }

    _isLoading = false;
    notifyListeners();
  }
}
# The final step is to run the app
flutter run
