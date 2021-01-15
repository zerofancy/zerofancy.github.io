title: android项目开发：持久化
author: 归零幻想
publishDate: 2020-12-24
editDate: 2020-12-28
tags: [安卓, kotlin, 第一行代码, 笔记]

<!--config-->

《第一行代码》阅读记录，有关数据持久化存储，略过了数据库的方式。

# 持久化
## 文件存储

先暂时只是写了个demo，有需要再深入看。

```kotlin

    private fun saveText(inputText: String) {
        try {
            val output = openFileOutput("data", Context.MODE_PRIVATE)
            val writer = BufferedWriter(OutputStreamWriter(output))
            writer.use {
                it.write(inputText)
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }

    private fun loadText(): String {
        val content = StringBuilder()
        try {
            val input = openFileInput("data")
            val reader = BufferedReader(InputStreamReader(input))
            reader.use { r ->
                r.forEachLine {
                    content.appendLine(it)
                }
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
        return content.toString()
    }
```

## SharedPreference

```kotlin

            saveButton.setOnClickListener {
                getSharedPreferences("data", Context.MODE_PRIVATE).edit {
                    putString("name", "Tom")
                    putInt("age", 28)
                    putBoolean("married", false)
                }
            }

            restoreButton.setOnClickListener {
                getSharedPreferences("data", Context.MODE_PRIVATE).apply {
                    val name = getString("name", "苏珊")
                    val age = getInt("age", 15)
                    val isMarried = getBoolean("married", false)
                    Toast.makeText(
                        this@MainActivity,
                        "${name}年龄${age}岁${if (isMarried) "已婚" else "未婚"}",
                        Toast.LENGTH_LONG
                    ).show()
                }
            }

```

## 数据库


基本用法demo

```kotlin

package top.ntutn.databasetest

import android.content.ContentValues
import android.content.Context
import android.database.Cursor
import android.database.sqlite.SQLiteDatabase
import android.database.sqlite.SQLiteOpenHelper
import android.util.Log
import android.widget.Toast

class MyDatabaseHelper(val context: Context, name: String, version: Int) :
    SQLiteOpenHelper(context, name, null, version) {
    private val createBook = """
        create table Book (
            id integer primary key autoincrement,
            author text,
            price real,
            pages integer,
            name text
        )
    """.trimIndent()
    private val createCategory = """
        create table Category (
            id integer primary key autoincrement,
            category_name text,
            category_code integer
        )
    """.trimIndent()

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(createBook)
        db.execSQL(createCategory)
        Toast.makeText(context, "建立数据库完成", Toast.LENGTH_LONG).show()
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        if (oldVersion == newVersion) return
        Log.d(javaClass.simpleName, "旧版是${oldVersion}，新版是${newVersion}，数据库升级开始")
        db.execSQL("drop table if exists Book")
        db.execSQL("drop table if exists Category")
        onCreate(db)
    }
}

fun cvOf(vararg pairs: Pair<String, Any?>): ContentValues {
    val cv = ContentValues()
    for (pair in pairs) {
        val key = pair.first
        when (val value = pair.second) {
            is Int -> cv.put(key, value)
            is Long -> cv.put(key, value)
            is Short -> cv.put(key, value)
            is Float -> cv.put(key, value)
            is Double -> cv.put(key, value)
            is Boolean -> cv.put(key, value)
            is String -> cv.put(key, value)
            is Byte -> cv.put(key, value)
            is ByteArray -> cv.put(key, value)
            null -> cv.putNull(key)
            else -> throw IllegalArgumentException("不支持的cv类型")
        }
    }
    return cv
}

fun Cursor.getString(name: String): String = getString(getColumnIndex(name))

fun Cursor.getInt(name: String): Int = getInt(getColumnIndex(name))

fun Cursor.getDouble(name: String): Double = getDouble(getColumnIndex(name))
```

```kotlin

package top.ntutn.databasetest

import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import top.ntutn.databasetest.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 6)

        binding.createDatabaseButton.setOnClickListener {
            dbHelper.writableDatabase
        }
        binding.addDataButton.setOnClickListener {
            val db = dbHelper.writableDatabase
            val values1 = cvOf(
                "name" to "The Da Vinci Code",
                "author" to "Dan Brown",
                "pages" to 454,
                "price" to 16.96
            )
            db.insert("Book", null, values1)
            val values2 = cvOf(
                "name" to "The Lost Symbol",
                "author" to "Dan Brown",
                "pages" to 510,
                "price" to 19.95
            )
            db.insert("Book", null, values2)
        }
        binding.updateDataButton.setOnClickListener {
            val db = dbHelper.writableDatabase
            val values = cvOf("price" to 10.99)
            db.update("Book", values, "name=?", arrayOf("The Da Vinci Code"))
        }
        binding.deleteDataButton.setOnClickListener {
            val db = dbHelper.writableDatabase
            db.delete("Book", "pages > ?", arrayOf("500"))
        }
        binding.queryDataButton.setOnClickListener {
            val db = dbHelper.writableDatabase
            val cursor = db.query("Book", null, null, null, null, null, null)
            if (cursor.moveToFirst()) {
                do {
                    val name = cursor.getString("name")
                    val author = cursor.getString("author")
                    val pages = cursor.getInt("pages")
                    val price = cursor.getDouble("price")
                    Log.d(
                        javaClass.simpleName,
                        "name: $name, author: $author, pages: $pages, price: $price"
                    )
                } while (cursor.moveToNext())
            }
            cursor.close()
        }
    }
}
```

<!--summary-->
