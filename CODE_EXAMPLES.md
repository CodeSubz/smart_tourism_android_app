# 🔍 Code Examples - How the Smart Tourism App Works

This document shows key code snippets that demonstrate the core functionality of the Smart Tourism app.

## 🤖 AI-Powered Monument Recognition

### Camera Activity (Android)
The main camera recognition logic in `CameraActivity.java`:

```java
public abstract class CameraActivity extends AppCompatActivity 
        implements OnImageAvailableListener, Camera.PreviewCallback {
    
    // Recognition threshold - when confidence is above this, show monument info
    private static final float RECOGNITION_THRESHOLD = 1.6f;
    
    // Process each camera frame for monument recognition
    @Override
    public void onImageAvailable(ImageReader reader) {
        Image image = null;
        try {
            image = reader.acquireLatestImage();
            if (image == null) return;
            
            // Convert camera image to bitmap for processing
            Bitmap bitmap = imageToBitmap(image);
            
            // Run AI classification on the image
            runInBackground(() -> {
                final List<Recognition> results = classifier.recognizeImage(bitmap);
                
                runOnUiThread(() -> {
                    showResultsInBottomSheet(results);
                    showFrameInfo(bitmap.getWidth() + "x" + bitmap.getHeight());
                });
            });
            
        } catch (Exception e) {
            LOGGER.e(e, "Exception in onImageAvailable");
        } finally {
            if (image != null) {
                image.close();
            }
        }
    }
    
    // Show monument information when recognized
    private void showResultsInBottomSheet(List<Recognition> results) {
        if (results != null && results.size() >= 1) {
            Recognition recognition = results.get(0);
            
            // Check if confidence is high enough
            if (recognition.getConfidence() > RECOGNITION_THRESHOLD) {
                // Show monument popup with information
                showMonumentPopup(recognition.getTitle());
            }
        }
    }
}
```

## 🏗️ Database Creation (Python Backend)

### Main Database Builder
The Python script that processes images and creates the SQLite database:

```python
# build_sqlite.py - Core database creation
import numpy as np
from sklearn.model_selection import train_test_split
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
import sqlite3
import cv2
import os

# Neural network models available for processing
types = [
    ('MobileNetV3_Large_100', 'models/.../mobilenet_v3_large_100_224.tflite'),
    ('MobileNetV3_Large_075', 'models/.../mobilenet_v3_large_075_224.tflite'),
    ('MobileNetV3_Small_100', 'models/.../mobilenet_v3_small_100_224.tflite')
]

def check_guides_and_images(guides_list, monuments_images_list):
    """Ensure every monument has both images and guide content"""
    missing_guides = monuments_images_list - guides_list
    missing_images = guides_list - monuments_images_list
    
    if missing_guides:
        print(f"[WARN] Monuments without guides: {missing_guides}")
    if missing_images:
        print(f"[WARN] Guides without images: {missing_images}")
    
    if not missing_guides and not missing_images:
        print("[INFO] Guides and monuments are consistent")

def process_monument_images(dataset_path, city_name):
    """Extract features from monument images using neural networks"""
    monuments = []
    features = []
    
    for monument_dir in os.listdir(dataset_path):
        monument_path = os.path.join(dataset_path, monument_dir)
        if not os.path.isdir(monument_path):
            continue
            
        print(f"Processing {monument_dir}...")
        
        # Process all images for this monument
        monument_features = []
        for image_file in os.listdir(monument_path):
            if image_file.lower().endswith(('.jpg', '.jpeg', '.png')):
                image_path = os.path.join(monument_path, image_file)
                
                # Load and preprocess image
                image = cv2.imread(image_path)
                image = cv2.resize(image, (224, 224))  # Standard size
                image = image / 255.0  # Normalize
                
                # Extract features using neural network
                feature_vector = extract_features(image)
                monument_features.append(feature_vector)
        
        # Average features for this monument
        if monument_features:
            avg_features = np.mean(monument_features, axis=0)
            monuments.append(monument_dir)
            features.append(avg_features)
    
    return monuments, features

def create_sqlite_database(monuments, features, guides, city_name):
    """Create SQLite database with all monument data"""
    db_path = f"models/src/main/assets/databases/{city_name}.db"
    
    # Create database connection
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    # Create tables
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS monuments (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            features BLOB NOT NULL,
            guide_content TEXT,
            latitude REAL,
            longitude REAL,
            categories TEXT,
            attributes TEXT
        )
    ''')
    
    # Insert monument data
    for i, monument in enumerate(monuments):
        feature_blob = features[i].tobytes()  # Convert numpy array to bytes
        guide_text = guides.get(monument, '')
        
        cursor.execute('''
            INSERT INTO monuments 
            (name, features, guide_content, latitude, longitude, categories, attributes)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        ''', (monument, feature_blob, guide_text, 0.0, 0.0, '', ''))
    
    conn.commit()
    conn.close()
    print(f"Database created: {db_path}")
```

## 📱 Android Monument List (Main Activity)

### Loading and Displaying Monuments
How the app shows monuments to users:

```java
// MainActivity.java - Main monument listing
public class MainActivity extends AppCompatActivity {
    
    private RecyclerView monumentsRecyclerView;
    private MonumentAdapter monumentAdapter;
    private DatabaseAccess databaseAccess;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize database access
        databaseAccess = DatabaseAccess.getInstance(this);
        
        // Setup RecyclerView for monument list
        monumentsRecyclerView = findViewById(R.id.monuments_recycler_view);
        monumentsRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        
        // Load and display monuments
        loadMonuments();
        
        // Setup camera button
        FloatingActionButton cameraFab = findViewById(R.id.camera_fab);
        cameraFab.setOnClickListener(v -> openCamera());
    }
    
    private void loadMonuments() {
        // Get user preferences for filtering
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
        String preferredCategories = prefs.getString("preferred_categories", "");
        
        // Load monuments from database
        databaseAccess.open();
        List<Monument> monuments = databaseAccess.getMonumentsByPreferences(
            preferredCategories, 
            getCurrentLocation()
        );
        databaseAccess.close();
        
        // Setup adapter with click handling
        monumentAdapter = new MonumentAdapter(monuments, this);
        monumentsRecyclerView.setAdapter(monumentAdapter);
    }
    
    private void openCamera() {
        // Check camera permission
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) 
            == PackageManager.PERMISSION_GRANTED) {
            
            Intent cameraIntent = new Intent(this, CameraActivity.class);
            startActivity(cameraIntent);
        } else {
            // Request camera permission
            ActivityCompat.requestPermissions(this, 
                new String[]{Manifest.permission.CAMERA}, 
                CAMERA_PERMISSION_REQUEST);
        }
    }
    
    @Override
    public void onMonumentClick(Monument monument) {
        // Open guide for selected monument
        Intent guideIntent = new Intent(this, GuideActivity.class);
        guideIntent.putExtra("monument_name", monument.getName());
        guideIntent.putExtra("monument_id", monument.getId());
        startActivity(guideIntent);
    }
}
```

## 📖 Guide Viewer with Recommendations

### Displaying Rich Monument Information
How guides are shown with recommendations:

```java
// GuideActivity.java - Monument guide viewer
public class GuideActivity extends AppCompatActivity {
    
    private MarkdownView markdownView;
    private RecyclerView recommendationsRecyclerView;
    private DatabaseAccess databaseAccess;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_guide);
        
        String monumentName = getIntent().getStringExtra("monument_name");
        
        // Initialize views
        markdownView = findViewById(R.id.markdown_view);
        recommendationsRecyclerView = findViewById(R.id.recommendations_recycler_view);
        
        // Load and display guide content
        loadGuideContent(monumentName);
        
        // Load recommendations
        loadRecommendations(monumentName);
        
        // Setup map button
        Button mapButton = findViewById(R.id.open_map_button);
        mapButton.setOnClickListener(v -> openMap(monumentName));
    }
    
    private void loadGuideContent(String monumentName) {
        databaseAccess = DatabaseAccess.getInstance(this);
        databaseAccess.open();
        
        // Get guide content from database
        String guideContent = databaseAccess.getGuideContent(monumentName);
        String markdownContent = processMarkdownContent(guideContent);
        
        // Display in markdown viewer
        markdownView.loadMarkdown(markdownContent);
        
        databaseAccess.close();
    }
    
    private void loadRecommendations(String currentMonument) {
        databaseAccess.open();
        
        // Get similar monuments based on categories and user preferences
        List<Monument> recommendations = databaseAccess.getRecommendations(
            currentMonument,
            getUserPreferences(),
            3  // Number of recommendations
        );
        
        databaseAccess.close();
        
        // Setup recommendations adapter
        RecommendationAdapter adapter = new RecommendationAdapter(
            recommendations, 
            monument -> {
                // When recommendation clicked, open its guide
                Intent intent = new Intent(this, GuideActivity.class);
                intent.putExtra("monument_name", monument.getName());
                startActivity(intent);
            }
        );
        
        recommendationsRecyclerView.setAdapter(adapter);
    }
    
    private void openMap(String monumentName) {
        // Get monument coordinates
        databaseAccess.open();
        Monument monument = databaseAccess.getMonument(monumentName);
        databaseAccess.close();
        
        // Open in Google Maps
        Uri mapUri = Uri.parse(String.format(
            "geo:%f,%f?q=%f,%f(%s)",
            monument.getLatitude(),
            monument.getLongitude(),
            monument.getLatitude(),
            monument.getLongitude(),
            monument.getName()
        ));
        
        Intent mapIntent = new Intent(Intent.ACTION_VIEW, mapUri);
        mapIntent.setPackage("com.google.android.apps.maps");
        startActivity(mapIntent);
    }
}
```

## 🗄️ Database Helper Class

### Accessing Monument Data
The database access layer that connects to SQLite:

```java
// DatabaseAccess.java - Database operations
public class DatabaseAccess {
    
    private SQLiteOpenHelper openHelper;
    private SQLiteDatabase database;
    private static DatabaseAccess instance;
    
    private DatabaseAccess(Context context) {
        this.openHelper = new DatabaseOpenHelper(context);
    }
    
    public static DatabaseAccess getInstance(Context context) {
        if (instance == null) {
            instance = new DatabaseAccess(context);
        }
        return instance;
    }
    
    public void open() {
        this.database = openHelper.getReadableDatabase();
    }
    
    public void close() {
        if (database != null) {
            this.database.close();
        }
    }
    
    public List<Monument> getMonumentsByPreferences(String categories, Location userLocation) {
        List<Monument> monuments = new ArrayList<>();
        
        String query = "SELECT * FROM monuments";
        if (!categories.isEmpty()) {
            query += " WHERE categories LIKE ?";
        }
        query += " ORDER BY name";
        
        Cursor cursor = categories.isEmpty() ? 
            database.rawQuery(query, null) :
            database.rawQuery(query, new String[]{"%" + categories + "%"});
        
        if (cursor.moveToFirst()) {
            do {
                Monument monument = new Monument();
                monument.setId(cursor.getInt("id"));
                monument.setName(cursor.getString("name"));
                monument.setLatitude(cursor.getDouble("latitude"));
                monument.setLongitude(cursor.getDouble("longitude"));
                monument.setCategories(cursor.getString("categories"));
                
                // Calculate distance if user location available
                if (userLocation != null) {
                    float distance = calculateDistance(
                        userLocation.getLatitude(),
                        userLocation.getLongitude(),
                        monument.getLatitude(),
                        monument.getLongitude()
                    );
                    monument.setDistance(distance);
                }
                
                monuments.add(monument);
            } while (cursor.moveToNext());
        }
        
        cursor.close();
        
        // Sort by distance if location available
        if (userLocation != null) {
            Collections.sort(monuments, (m1, m2) -> 
                Float.compare(m1.getDistance(), m2.getDistance()));
        }
        
        return monuments;
    }
    
    public String getGuideContent(String monumentName) {
        String content = "";
        
        Cursor cursor = database.rawQuery(
            "SELECT guide_content FROM monuments WHERE name = ?",
            new String[]{monumentName}
        );
        
        if (cursor.moveToFirst()) {
            content = cursor.getString("guide_content");
        }
        
        cursor.close();
        return content;
    }
    
    public List<Monument> getRecommendations(String currentMonument, 
                                           UserPreferences preferences, 
                                           int limit) {
        // Complex recommendation logic based on:
        // - Similar categories
        // - User preferences
        // - Proximity
        // - Previous interactions
        
        String query = """
            SELECT m.*, 
                   (CASE WHEN m.categories LIKE ? THEN 1 ELSE 0 END) as category_match,
                   ABS(m.latitude - ?) + ABS(m.longitude - ?) as distance_score
            FROM monuments m 
            WHERE m.name != ?
            ORDER BY category_match DESC, distance_score ASC
            LIMIT ?
        """;
        
        // Get current monument location for proximity calculation
        Monument current = getMonument(currentMonument);
        
        Cursor cursor = database.rawQuery(query, new String[]{
            "%" + preferences.getPreferredCategory() + "%",
            String.valueOf(current.getLatitude()),
            String.valueOf(current.getLongitude()),
            currentMonument,
            String.valueOf(limit)
        });
        
        List<Monument> recommendations = new ArrayList<>();
        if (cursor.moveToFirst()) {
            do {
                Monument monument = cursorToMonument(cursor);
                recommendations.add(monument);
            } while (cursor.moveToNext());
        }
        
        cursor.close();
        return recommendations;
    }
}
```

## 🧠 Key Features Explained

### 1. **Real-time AI Recognition**
- Camera captures frames continuously
- Each frame is processed by TensorFlow Lite model
- When confidence exceeds threshold → show monument popup
- All processing happens on-device (offline)

### 2. **Smart Recommendations**
- Based on monument categories and attributes
- Consider user's location and preferences
- Learn from user interactions over time
- Show 3 most relevant monuments

### 3. **Offline-First Architecture**
- All data stored in local SQLite database
- Guides cached as Markdown text
- No internet required for core functionality
- Images and features pre-processed

### 4. **Personalization Engine**
- User sets preferred categories (Art, History, Architecture)
- App learns from monument visits
- Recommendations improve over time
- Location-aware suggestions

This code shows how the app seamlessly combines computer vision, local databases, and user preferences to create an intelligent tourism experience!