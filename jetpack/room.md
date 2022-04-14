### Room数据库使用
官方使用文档地址 https://developer.android.google.cn/training/data-storage/room?hl=zh-cn
使用步骤官方文档写的很清楚，这里简单记录下，让自己更加熟悉。

#### 添加依赖
```
dependencies {
    def room_version = "2.4.2"
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    // optional - RxJava2 support for Room
    implementation "androidx.room:room-rxjava2:$room_version"
    // optional - RxJava3 support for Room
    implementation "androidx.room:room-rxjava3:$room_version"
    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation "androidx.room:room-guava:$room_version"
    // optional - Test helpers
    testImplementation "androidx.room:room-testing:$room_version"
    // optional - Paging 3 Integration
    implementation "androidx.room:room-paging:2.5.0-alpha01"

    kapt "androidx.room:room-compiler:2.4.2"
}
```

#### 数据库实体类（Entity）

我用的是github上的接口来做demo程序的，这里github的返回字段把自己坑了，返回字段中有个private字段，java中private是关键字，导致自动生成的代码中，private字段会自动改名，然后编译时候就会报错，报错内容如下，
>>> Entities and POJOs must have a usable public constructor. You can have an empty constructor or a constructor whose parameters match the fields (by name and type).
public final class GitHubRepo {
             ^
  Tried the following constructors but they failed to match:
  GitHubRepo(int,java.lang.String,java.lang.String,java.lang.Boolean) -> [param:id -> matched field:id, param:node_id -> matched field:node_id, param:full_name -> matched field:full_name, param:p3 -> matched field:unmatched]

粗看错误信息，Entities and POJOs must have a usable public constructor. You can have an empty constructor or a constructor whose parameters match the fields (by name and type). 
提示需要增加一个constructor, 我就按照提示增加了一个constructor，内容如下，增加后，确实可以使用，但看着代码总感觉很奇怪，如果有较多参数时，constructor中内容看着不优雅， 翻看Google官方提供的示例中，也没有额外的增加constructor，最后发现问题在private参数上，
```
@Entity(tableName = "github_repo")
data class GitHubRepo(
//    @PrimaryKey(autoGenerate = true) val Ida: Int = 0,
    @PrimaryKey(autoGenerate = true) var id: Int = 0,
    var node_id: String,
    var full_name: String,
    var private: Boolean

    ) {
    constructor(): this(0, "", "", false)
}
```

正确做法如下，将private字段改名成非关键字，然后分别增加```@ColumnInfo(name = "private")```表示在数据库中存的字段名，
   ```@SerializedName("private")```表示在json解析时候的字段，内容如下
   
```
@Entity(tableName = "github_repo")
data class GitHubRepo(
    @PrimaryKey(autoGenerate = true) var id: Int = 0,
    var node_id: String,
    var full_name: String,
    
    @ColumnInfo(name = "private")
    @SerializedName("private")
    //github的这个字段名称把自己坑了，private是关键字
    var _private: Boolean,
    
//    @ColumnInfo(defaultValue = "")
//    var name: String,
    )
```


#### 数据访问对象DAO（data access object）

```
@Dao
interface GitHubRepoDao {
    @Query("SELECT * FROM github_repo")
    fun getAll(): List<GitHubRepo>

    @Query("SELECT * FROM github_repo WHERE id IN (:ids)")
    fun loadAllByIds(ids: IntArray): List<GitHubRepo>
//
    @Query("SELECT * FROM github_repo WHERE full_name = :full_name")
    fun findByName(full_name: String): List<GitHubRepo>
//
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertAll(vararg repos: GitHubRepo)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertAll2(repos: List<GitHubRepo>)

}
```

查询时候也可以返回一个Flow，方便后续操作
```
@Query("SELECT * FROM github_repo")
fun getAll(): Flow<List<GitHubRepo>>
```

#### 定义一个数据库文件
```
@Database(
    entities = [User::class, GitHubRepo::class],
    version = 1,
    exportSchema = true,
    autoMigrations = [AutoMigration(from = 1, to = 2)])
    )
abstract class AppDatabase: RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun githubRepoDao(): GitHubRepoDao

    companion object {

        @Volatile private var instance: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return instance ?: synchronized(this){
                instance ?: buildDatabase(context).also {
                    instance = it
                }

            }
        }

        private fun buildDatabase(context: Context): AppDatabase {
            return Room.databaseBuilder(context, AppDatabase::class.java, DATABASE_NAME).build()
        }
    }
}

private const val DATABASE_NAME = "xzl_common_db"
```
Room 在 2.4.0-alpha01 及更高版本中支持自动迁移，直接更新数据库版本，会丢失数据，做数据迁移需要增加部分内容，build.gradle中增加配置内容，
```
version = 3,
exportSchema = true,
autoMigrations = [AutoMigration(from = 1, to = 2), AutoMigration(from = 2, to = 3)]


android {
    ...

    defaultConfig {
        ...
        kapt {
            arguments {
                arg("room.schemaLocation", "$projectDir/schemas")
            }
        }
    }

    sourceSets {
        getByName("androidTest") {
            assets.srcDirs(files(projectDir, "schemas"))
        }
    }
    ...
```

#### 使用数据库
```
    private val githubRepoDao = AppDatabase.getInstance(application).githubRepoDao()
    
    suspend fun localRepos(): List<GitHubRepo> = withContext(Dispatchers.IO) {
        githubRepoDao.getAll()
    }
```




























