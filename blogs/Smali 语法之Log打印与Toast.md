# Smali语法 log打印 与 Toast

```
public void a(Uri u){
    u = Uri.parse("bugle://avatar/l");

    if(u != null){
        android.util.Log.e("SHUIYES", u.toString());
    }

    if(u == null){
        Toast.makeText(this, "text", 0).show();
    }
}
```

```
# virtual methods
.method public a(Landroid/net/Uri;)V
    .locals 2
    .param p1, "u"    # Landroid/net/Uri;

    .prologue
    .line 19
    const-string v0, "bugle://avatar/l"

    invoke-static {v0}, Landroid/net/Uri;->parse(Ljava/lang/String;)Landroid/net/Uri;

    move-result-object p1

    .line 21
    if-eqz p1, :cond_0

    .line 22
    const-string v0, "SHUIYES"

    invoke-virtual {p1}, Landroid/net/Uri;->toString()Ljava/lang/String;

    move-result-object v1

    invoke-static {v0, v1}, Landroid/util/Log;->e(Ljava/lang/String;Ljava/lang/String;)I

    .line 24
    :cond_0
    if-nez p1, :cond_1

    .line 25
    const-string v0, "text"

    const/4 v1, 0x0

    invoke-static {p0, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;

    move-result-object v0

    invoke-virtual {v0}, Landroid/widget/Toast;->show()V

    .line 27
    :cond_1
    return-void
.end method
```
