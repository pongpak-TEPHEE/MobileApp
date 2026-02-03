# MobileApp
เก็บชิ้นงานในรายวิชา Mobile application developer

3/2/69
# file MainActivity.kt ----------------------------------------------------------------------------------------------------------------------------------------------------------|
package com.example.shopapp


import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Edit
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Menu
import androidx.compose.material.icons.filled.Search
import androidx.compose.material.icons.filled.ShoppingCart
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.NavigationBar
import androidx.compose.material3.NavigationBarItem
import androidx.compose.material3.NavigationBarItemDefaults
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.RadioButton
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.material3.TopAppBar
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.TextFieldValue
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavController
import androidx.navigation.NavHostController
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.example.shopapp.ui.theme.OrderViewModel
import com.example.shopapp.ui.theme.OrderViewModelFactory
import com.example.shopapp.ui.theme.ShopAppTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ShopAppTheme {
                MyApp()
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyApp() {
    val navController = rememberNavController()
    val items = listOf("main", "Search", "history")
    var selectedItem by remember { mutableIntStateOf(0) }
    val icons = listOf(Icons.Default.Home, Icons.Default.Search, Icons.Default.Menu)
    val context = LocalContext.current
    val viewModel: OrderViewModel = viewModel(
        factory = OrderViewModelFactory(context)
    )


    Scaffold(
        topBar = {
            TopAppBar(
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = Color(0xFF6E5034),
                    titleContentColor = Color.White
                ),
                title = { Text("Shop App") },
                actions = {
                    IconButton(onClick = {}) {
                        Icon(Icons.Default.ShoppingCart, contentDescription = null, tint = Color.White)
                    }
                }
            )
        },
        bottomBar = {
            NavigationBar(containerColor = Color(0xFF6E5034)) {
                items.forEachIndexed { index, item ->
                    NavigationBarItem(
                        icon = { Icon(icons[index], contentDescription = item) },
                        label = { Text(item) },
                        selected = selectedItem == index,
                        onClick = {
                            selectedItem = index
                            when (index) {
                                0 -> navController.navigate("main") {
                                    popUpTo("main") { saveState = true }
                                    launchSingleTop = true
                                    restoreState = true
                                }
                                1 -> navController.navigate("search") {
                                    popUpTo("main") { saveState = true }
                                    launchSingleTop = true
                                    restoreState = true
                                }
                                2 -> navController.navigate("history") {
                                    popUpTo("main") { saveState = true }
                                    launchSingleTop = true
                                    restoreState = true
                                }
                            }
                        },
                        colors = NavigationBarItemDefaults.colors(
                            selectedIconColor = Color.White,
                            unselectedIconColor = Color.Gray,
                            indicatorColor = Color(0xFFFFC107)
                        )
                    )
                }
            }
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = "main",
            modifier = Modifier.padding(innerPadding)
        ) {
            composable("main") {
                Greeting(navController = navController)
            }

            composable("search") {
                SearchScreen()
            }

            composable("history") {
                HistoryScreen(viewModel, navController, modifier = Modifier)
            }

            composable(
                route = "summary/{size}/{count}/{detail}",
                arguments = listOf(
                    navArgument("size") { type = NavType.StringType },
                    navArgument("count") { type = NavType.IntType },
                    navArgument("detail") { type = NavType.StringType }
                )
            ) { backStackEntry ->
                val size = backStackEntry.arguments?.getString("size") ?: ""
                val count = backStackEntry.arguments?.getInt("count") ?: 0
                val detail = backStackEntry.arguments?.getString("detail") ?: ""

                SummaryScreen(size, count, detail, navController, viewModel)
            }

            composable(
                route = "update/{id}",
                arguments = listOf(
                    navArgument("id"){ type = NavType.IntType }
                    )
            ) { backStackEntry ->
                val id = backStackEntry.arguments?.getInt("id") ?: 0

                UpdateScreen(orderID = id, viewModel = viewModel)
            }
        }
    }
}





@Composable
fun SearchScreen() {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Icon(
            imageVector = Icons.Default.Search,
            contentDescription = null,
            modifier = Modifier.size(100.dp),
            tint = Color.Gray
        )
        Text("หน้าค้นหาสินค้า", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Text("เร็วๆ นี้...", color = Color.Gray)
    }
}





@Composable
fun Greeting(navController: androidx.navigation.NavController, modifier: Modifier = Modifier) {
    val radioOption = listOf("8 US", "9 US", "9.5 US", "10 US")
    var selectedOption by remember { mutableStateOf(radioOption[0]) }
    var passText by remember { mutableStateOf(TextFieldValue("")) }
    var count by remember { mutableIntStateOf(0) }

    Column(modifier = modifier.fillMaxSize()) {
        Row(modifier = Modifier.fillMaxWidth().padding(top = 16.dp), horizontalArrangement = Arrangement.Center) {
            Image(
                painter = painterResource(R.drawable.adidas_pro_4),
                contentDescription = "adidas",
                modifier = Modifier.width(350.dp)
            )
        }

        Text(text = "Adidas Adizero Pro 4", modifier = Modifier.padding(15.dp), fontWeight = FontWeight.Bold, fontSize = 28.sp)

        Row(modifier = Modifier.fillMaxWidth()) {
            radioOption.forEach { option ->
                Row(verticalAlignment = Alignment.CenterVertically) {
                    RadioButton(selected = selectedOption == option, onClick = { selectedOption = option })
                    Text(option)
                }
            }
        }

        Text("รายละเอียดเพิ่มเติม : ", modifier = Modifier.padding(15.dp))
        OutlinedTextField(
            modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp),
            value = passText,
            onValueChange = { passText = it },
            placeholder = { Text("เช่น มีตำหนิไหม") }
        )

        Text(text = "จำนวน:", modifier = Modifier.padding(15.dp))
        Row(modifier = Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.Center, verticalAlignment = Alignment.CenterVertically) {
            IconButton(onClick = { if (count > 0) count-- }) {
                Icon(painterResource(R.drawable.do_not_disturb), contentDescription = "ลบ", modifier = Modifier.size(24.dp))
            }
            Text(text = count.toString(), modifier = Modifier.padding(horizontal = 16.dp), fontSize = 20.sp, fontWeight = FontWeight.Bold)
            IconButton(onClick = { count++ }) {
                Icon(painterResource(R.drawable.add_circle), contentDescription = "เพิ่ม", modifier = Modifier.size(24.dp))
            }
        }

        Button(
            modifier = Modifier.fillMaxWidth().padding(16.dp),
            onClick = {
                val detail =  if (passText.text.isEmpty()) "ไม่มี" else passText.text.replace("/", "-")
                // ส่งข้อมูลไปหน้า summary
                navController.navigate("summary/${selectedOption}/${count}/${detail}")
            }
        ) {
            Text("ดูสรุปรายละเอียด")
        }
    }
}





@Composable
fun HistoryScreen(viewModel: OrderViewModel, navController: NavHostController, modifier: Modifier) {

    val order by viewModel.orders.collectAsState(initial = emptyList())

    Column(modifier = modifier.fillMaxSize().padding(16.dp)) {
        Text("ประวัติการสั่งซื้อ", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(8.dp))

        var confirmDialog by remember { mutableStateOf(false)}
        LazyColumn {
            items(order) { orders ->
                androidx.compose.material3.Card(
                    modifier = Modifier.fillMaxWidth().padding(vertical = 4.dp)
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text("Size: ${orders.size}", fontWeight = FontWeight.Bold)
                        Text("Quantity: ${orders.qty}")
                        Text("Detail: ${orders.detail}", color = Color.Gray)
                    }
                Row {
                    IconButton(onClick = {
                        navController.navigate("update/${orders.id}")
                    }) {
                        Icon(imageVector = Icons.Default.Edit, contentDescription = null)
                    }
                    if (confirmDialog){
                        AlertDialog(
                            onDismissRequest = {confirmDialog = false},
                            title = {Text("confirm to delete")},
                            text = {Text("You are confirm to delete this order")},
                            confirmButton = {
                                    TextButton(onClick = {
                                        confirmDialog = false
                                        viewModel.deleteOrder(orders)
                                }) {Text("Delete")}
                            },
                            dismissButton = {
                                TextButton(onClick = {confirmDialog = false}) {Text("Cancel") }
                            }
                            )
                    }
                    IconButton(onClick = { confirmDialog = true}) {
                        Icon(imageVector = Icons.Default.Delete, contentDescription = null)
                    }
                }
                }
            }
        }
    }
}

# file Order.ky ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
package com.example.shopapp

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.shopapp.ui.theme.OrderViewModel


@Composable
fun SummaryScreen(size: String, count: Int, detail: String, navController: androidx.navigation.NavController, viewModel: OrderViewModel) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("สรุปข้อมูลการสั่งซื้อ", fontSize = 24.sp, fontWeight = FontWeight.Bold)

        Spacer(modifier = Modifier.height(20.dp))

        Column(modifier = Modifier.fillMaxWidth()) {
            Text("ไซส์: $size", fontSize = 18.sp)
            Text("จำนวน: $count คู่", fontSize = 18.sp)
            Text("รายละเอียด: $detail", fontSize = 18.sp)
        }

        Spacer(modifier = Modifier.height(40.dp))
        Button(
            onClick = { navController.popBackStack() },
            modifier = Modifier.fillMaxWidth().padding(bottom = 8.dp)
        ) {
            Text("ย้อนกลับไปแก้ไข")
        }

        Button(
            onClick = {
                viewModel.insertOrder(
                    size = size, detail = detail, qty = count
                )
            },
            modifier = Modifier.fillMaxWidth().padding(bottom = 8.dp)
        ) {
            Text("OK")
        }

        Button(
            onClick = {
                navController.navigate("main") {
                    popUpTo("main") { inclusive = true }
                }
            },
            modifier = Modifier.fillMaxWidth(),
            colors = ButtonDefaults.buttonColors(containerColor = Color.Red) // ใช้สีแดงเพื่อให้เด่นว่าเป็นปุ่มลบ/ล้าง
        ) {
            Text("ล้างรายการสั่งซื้อ", color = Color.White)
        }
    }
}

# file db.kt -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
package com.example.shopapp.ui.theme

import android.content.Context
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import androidx.room.Dao
import androidx.room.Database
import androidx.room.Delete
import androidx.room.Entity
import androidx.room.Insert
import androidx.room.PrimaryKey
import androidx.room.Query
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.Update
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.launch


@Entity(tableName = "orders")
data class OrderEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val size: String,
    val detail: String?,
    val qty: Int
)

@Dao
interface OrderDao {
    @Insert
    suspend fun insert(order: OrderEntity)

    @Query("SELECT * FROM orders")
    fun getAll(): Flow<List<OrderEntity>> // list เพราะว่า

    @Query("SELECT * FROM orders WHERE id = :id")
    fun getByID(id: Int): Flow<OrderEntity?>

    @Update
    suspend fun update(order: OrderEntity)

    @Delete
    suspend fun delete(order: OrderEntity)
}

@Database(
    entities = [OrderEntity::class],
    version = 1
)
abstract class AppDatabase: RoomDatabase() {
    abstract fun orderDao(): OrderDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        fun getDatabase(context: Context): AppDatabase{
            return INSTANCE ?: synchronized(this){
                Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "order_db"
                ).build().also {
                    INSTANCE = it
                }
            }
        }
    }
}

class OrderRepository(private val dao: OrderDao) {
    suspend fun insert(order: OrderEntity){
        dao.insert(order)
    }

    suspend fun update(order: OrderEntity){
        dao.update(order)
    }

    suspend fun delete(order: OrderEntity) {
        dao.delete(order)
    }

    fun getByID(id: Int): Flow<OrderEntity?> {
        return dao.getByID(id)
    }
    val orders = dao.getAll()
}

class OrderViewModel(
    private val repository: OrderRepository
): ViewModel() {
    val orders = repository.orders
    fun insertOrder(size: String, detail: String?, qty: Int){
        viewModelScope.launch {
            repository.insert(order = OrderEntity(size = size, detail = detail, qty = qty))
        }
    }
    fun getOrderID(id: Int): Flow<OrderEntity?> {
        return repository.getByID(id)
    }

    fun updateOrder(id: Int, size: String, detail: String?, qty: Int){
        viewModelScope.launch {
            repository.update(OrderEntity(
                id = id,
                size = size,
                qty = qty,
                detail = detail
            ))
        }
    }

    fun deleteOrder(order: OrderEntity) {
        viewModelScope.launch {
            repository.delete((order))
        }
    }
}

class OrderViewModelFactory(context: Context): ViewModelProvider.Factory {
    private val dao = AppDatabase.getDatabase(context).orderDao()
    private val repository = OrderRepository(dao)

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(OrderViewModel::class.java)) {

            @Suppress("UNCHECKED_CAST")
            return OrderViewModel(repository) as T
        }
        throw IllegalArgumentException("ไม่พบ ViewModel ที่ต้องการ")
    }
}



# file Grable Scripts/build.gradle.kts (Module :app) ---------------------------------------------------------------------------------------------------------------------------------------------------------|
package com.example.shopapp

import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Edit
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Menu
import androidx.compose.material.icons.filled.Search
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.RadioButton
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavHostController
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.example.shopapp.ui.theme.OrderViewModel
import com.example.shopapp.ui.theme.OrderViewModelFactory

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun UpdateScreen(orderID: Int, viewModel: OrderViewModel, modifier: Modifier = Modifier) {

    val navController = rememberNavController()
    val items = listOf("main", "Search", "history")
    var selectedItem by remember { mutableIntStateOf(0) }
    val icons = listOf(Icons.Default.Home, Icons.Default.Search, Icons.Default.Menu)
    val context = LocalContext.current
    val viewModel: OrderViewModel = viewModel(
        factory = OrderViewModelFactory(context)
    )


    Scaffold() {
        NavHost(
            navController = navController,
            startDestination = "main"
        ) {
            composable("main") {
                Greeting_EditOrder(orderID, navController = navController, viewModel, modifier = Modifier)
            }

            composable("search") {
                SearchScreen_EditOrder()
            }

            composable("history") {
                HistoryScreen_EditOrder(viewModel, navController, modifier = Modifier)
            }

            composable(
                route = "summary/{size}/{count}/{detail}",
                arguments = listOf(
                    navArgument("size") { type = NavType.StringType },
                    navArgument("count") { type = NavType.IntType },
                    navArgument("detail") { type = NavType.StringType }
                )
            ) { backStackEntry ->
                val size = backStackEntry.arguments?.getString("size") ?: ""
                val count = backStackEntry.arguments?.getInt("count") ?: 0
                val detail = backStackEntry.arguments?.getString("detail") ?: ""

                SummaryScreen(size, count, detail, navController, viewModel)
            }

            composable(
                route = "update/{id}",
                arguments = listOf(
                    navArgument("id"){ type = NavType.IntType }
                )
            ) { backStackEntry ->
                val id = backStackEntry.arguments?.getInt("id") ?: 0

                UpdateScreen(orderID = id, viewModel = viewModel)
            }
        }
    }
}





@Composable
fun SearchScreen_EditOrder() {
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Icon(
            imageVector = Icons.Default.Search,
            contentDescription = null,
            modifier = Modifier.size(100.dp),
            tint = Color.Gray
        )
        Text("หน้าค้นหาสินค้า", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Text("เร็วๆ นี้...", color = Color.Gray)
    }
}





@Composable
fun Greeting_EditOrder(orderID: Int, navController: androidx.navigation.NavController, viewModel: OrderViewModel, modifier: Modifier) {
    val orders by viewModel.getOrderID(orderID).collectAsState(initial = null)

    val radioOption = listOf("8 US", "9 US", "9.5 US", "10 US")
    var selectedOption by remember { mutableStateOf(radioOption[0]) }
    var passText by remember { mutableStateOf("") }
    var count by remember { mutableIntStateOf(0) }

    LaunchedEffect(orders) {
        orders?.let{
            // ถ้าข้อมูลมีใน orders จะเอาข้อมูลนั้นมาใส่ใน orders
            passText = it.detail.toString()
            count = it.qty
            selectedOption = it.size
        }
    }

    Column(modifier = modifier.fillMaxSize()) {
        Row(modifier = Modifier.fillMaxWidth().padding(top = 16.dp), horizontalArrangement = Arrangement.Center) {
            Image(
                painter = painterResource(R.drawable.adidas_pro_4),
                contentDescription = "adidas",
                modifier = Modifier.width(350.dp)
            )
        }

        Text(text = "Adidas Adizero Pro 4", modifier = Modifier.padding(15.dp), fontWeight = FontWeight.Bold, fontSize = 28.sp)

        Row(modifier = Modifier.fillMaxWidth()) {
            radioOption.forEach { option ->
                Row(verticalAlignment = Alignment.CenterVertically) {
                    RadioButton(selected = selectedOption == option, onClick = { selectedOption = option })
                    Text(option)
                }
            }
        }

        Text("รายละเอียดเพิ่มเติม : ", modifier = Modifier.padding(15.dp))
        OutlinedTextField(
            modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp),
            value = passText,
            onValueChange = { passText = it },
            placeholder = { Text("เช่น มีตำหนิไหม") }
        )

        Text(text = "จำนวน:", modifier = Modifier.padding(15.dp))
        Row(modifier = Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.Center, verticalAlignment = Alignment.CenterVertically) {
            IconButton(onClick = { if (count > 0) count-- }) {
                Icon(painterResource(R.drawable.do_not_disturb), contentDescription = "ลบ", modifier = Modifier.size(24.dp))
            }
            Text(text = count.toString(), modifier = Modifier.padding(horizontal = 16.dp), fontSize = 20.sp, fontWeight = FontWeight.Bold)
            IconButton(onClick = { count++ }) {
                Icon(painterResource(R.drawable.add_circle), contentDescription = "เพิ่ม", modifier = Modifier.size(24.dp))
            }
        }

        Button(
            modifier = Modifier.fillMaxWidth().padding(16.dp),
            onClick = {
                 orders?.let {
                     viewModel.updateOrder(
                         id = orderID,
                         size = selectedOption,
                         qty = count,
                         detail = passText
                     )
                     navController.navigate("history")
                 }
            }
        ) {
            Text("แก้ไขข้อมูล")
        }
    }
}





@Composable
fun HistoryScreen_EditOrder(viewModel: OrderViewModel, navController: NavHostController, modifier: Modifier) {
    val order by viewModel.orders.collectAsState(initial = emptyList())

    Column(modifier = modifier.fillMaxSize().padding(16.dp)) {
        Text("ประวัติการสั่งซื้อ", fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Spacer(modifier = Modifier.height(8.dp))

        LazyColumn {
            items(order) { orders ->
                androidx.compose.material3.Card(
                    modifier = Modifier.fillMaxWidth().padding(vertical = 4.dp)
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text("Size: ${orders.size}", fontWeight = FontWeight.Bold)
                        Text("Quantity: ${orders.qty}")
                        Text("Detail: ${orders.detail}", color = Color.Gray)
                    }
                    Row {
                        IconButton(onClick = {
                            navController.navigate("update/${orders.id}")
                        }) {
                            Icon(imageVector = Icons.Default.Edit, contentDescription = null)
                        }
                        IconButton(onClick = {}) {
                            Icon(imageVector = Icons.Default.Delete, contentDescription = null)
                        }
                    }
                }
            }
        }
    }
}
