---------------MainActivity-----------------

package com.example.testapi

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.livedata.observeAsState
import androidx.compose.ui.Modifier
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import coil.compose.AsyncImage
import com.example.testapi.ui.theme.TestAPITheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            TestAPITheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
//                    ProductScreen(6)
                    AllProductsScreen()
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}


@Composable
fun ProductScreen(productId: Int, viewModel: ProductViewModel = viewModel( factory = ProductViewModelFactory(ProductRepository()))){
    val state = viewModel.product.observeAsState()
    LaunchedEffect(productId) { viewModel.loadProduct(productId)}
    when(val result = state.value) {
        is Resource.Loading -> {
            CircularProgressIndicator()
        }
        is Resource.Success -> {
            result.data?.let { ProductItem(product = it)}
        }
        else -> {
            print(message = "something is wrong")
        }
    }
}

@Composable
fun ProductItem(product: com.example.testapi.Product) {
    Card(modifier = Modifier.fillMaxWidth().padding(16.dp)) {
        Column(modifier = Modifier.padding(16.dp)) {
            AsyncImage(
                model = product.image,
                contentDescription = product.title,
                modifier = Modifier.fillMaxWidth().height(200.dp),
                contentScale = ContentScale.Fit
            )
            Text(product.title, fontWeight = FontWeight.Bold)
            Text("Price: $${product.price}")
            Text("Category: ${product.category}")
            Text("Rating: ${product.rating.rate} (${product.rating.count})")
        }
    }
}

@Composable
fun AllProductsScreen(viewModel: ProductViewModel = viewModel( factory = ProductViewModelFactory(ProductRepository()))){
    val state = viewModel.allProduct.observeAsState()
    LaunchedEffect(Unit) {viewModel.loadAllProducts() }
    when(val result = state.value) {
        is Resource.Loading -> {
            CircularProgressIndicator()
        }
        is Resource.Success -> {
            LazyColumn{
                items(result.data ?: emptyList()) { product -> ProductItem(product) }
            }
        }
        else -> {
            print(message = "something is wrong")
        }
    }
}


-------------------- build.gradle --------------------------

plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.example.testapi"
    compileSdk {
        version = release(36)
    }

    defaultConfig {
        applicationId = "com.example.testapi"
        minSdk = 33
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    kotlinOptions {
        jvmTarget = "11"
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.ui.graphics)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.compose.material3)
    implementation(libs.play.services.analytics.impl)
    implementation(libs.androidx.compose.runtime.livedata)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.compose.ui.test.junit4)
    debugImplementation(libs.androidx.compose.ui.tooling)
    debugImplementation(libs.androidx.compose.ui.test.manifest)
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")

    implementation("androidx.compose.runtime:runtime-livedata:1.7.5")
    implementation("androidx.lifecycle:lifecycle-livedata:2.8.7")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")

    implementation("io.coil-kt:coil-compose:2.6.0")
}


---------------------------- androidManifest.xml -----------------------------

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.TestAPI">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.TestAPI">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>



--------------------------- apiManagement.kt -------------------------------


package com.example.testapi


import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import androidx.lifecycle.viewmodel.compose.viewModel
import kotlinx.coroutines.launch
import okhttp3.Response
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import retrofit2.http.GET
import retrofit2.http.Path
import androidx.lifecycle.LiveData
import kotlin.properties.PropertyDelegateProvider

data class Product (
    val id: Int,
    val title: String,
    val price: Float,
    val description: String,
    val category: String,
    val image: String,
    val rating: Rating
)

data class Rating(
    val rate: Float,
    val count: Int
)

interface ApiServices{
    @GET("products/{id}")
    suspend fun getProductById(
        @Path("id") id: Int
    ): retrofit2.Response<Product>

    @GET("products")
    suspend fun getAllProducts(): retrofit2.Response<List<Product>>
}

object RetrofitInstance{
    private const val BASE_URL = "https://fakestoreapi.com/"
    val api: ApiServices by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiServices::class.java)
    }
}

sealed class Resource<T>(
    val data: T? = null,
    val message: String? = null
){
    class Success<T> (data: T): Resource<T>(data)
    class Error<T> (message: String?, data: T? = null): Resource<T>(data,message)
    class Loading<T>: Resource<T>()
}

class ProductRepository{
    suspend fun fetchProductById(id: Int): Resource<Product>{
        return try{
            val response = RetrofitInstance.api.getProductById(id)
            if(response.isSuccessful){
                response.body()?.let{
                    Resource.Success(data = it)
                }?: Resource.Error("Empty Body")
            }else{
                Resource.Error("Error ${response.code()}")
            }
        }catch(e: Exception){
            Resource.Error(e.message)
        }
    }

    suspend fun fetchAllProducts(): Resource<List<Product>>{
        return try{
            val response = RetrofitInstance.api.getAllProducts()
            if(response.isSuccessful){
                response.body()?.let{
                    Resource.Success(data = it)
                }?: Resource.Error("Empty Body")
            }else{
                Resource.Error("Error ${response.code()}")
            }
        }catch(e: Exception){
            Resource.Error(e.message)
        }
    }
}

class ProductViewModel(private val repository: ProductRepository): ViewModel(){
    private val _Product = MutableLiveData<Resource<Product>>()
    val product: LiveData<Resource<Product>> = _Product
    fun loadProduct(id:Int){
        _Product.value = Resource.Loading()
        viewModelScope.launch { _Product.value = repository.fetchProductById(id) }
    }

    private val _allProducts = MutableLiveData<Resource<List<Product>>>()
    val allProduct: LiveData<Resource<List<Product>>> = _allProducts
    fun loadAllProducts() {
        _allProducts.value = Resource.Loading()
        viewModelScope.launch { _allProducts.value = repository.fetchAllProducts() }
    }
}

class ProductViewModelFactory(
    private val repository: ProductRepository
): ViewModelProvider.Factory{
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if(modelClass.isAssignableFrom(ProductViewModel::class.java)){
            return ProductViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknow ViewModel class")
    }
}













