title: android项目开发：持久化
author: 归零幻想
publishDate: 2020-12-24
editDate: 2020-12-24
tags: [tag1, tag2]

<!--config-->

《第一行代码》阅读记录，有关数据持久化存储，略过了数据库的方式。

# 持久化
## 文件存储

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

略过。等有使用需求的时候再看。

<!--summary-->
