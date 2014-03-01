#ステップアップヒント

##真っ直ぐ進む

~~~
run(info)
  walkUp
end
~~~

##壁を避ける

~~~
run(info)
end
~~~

##敵にブロックを置く
上下左右に敵がいるかチェックしていればブロックを置く

~~~
run(info)
  if info[1] == TYPE_ENEMY
    putUp
  elsif info[3] == TYPE_ENEMY
    putLeft
  elsif info[5] == TYPE_ENEMY
    putRight
  elsif info[7] == TYPE_ENEMY
    putDown
  end
end
~~~

##アイテムを取る

~~~
run(info)
  if info[1] == TYPE_ITEM
    walkUp
  elsif info[3] == TYPE_ITEM
    walkLeft
  elsif info[5] == TYPE_ITEM
    walkRight
  elsif info[7] == TYPE_ITEM
    walkDown
  end
end
~~~

##壁がなければ進む

~~~
WALK_UP    = 1
WALK_LEFT  = 2
WALK_RIGHT = 3
WALK_DOWN  = 4

def safe_walk(info, direction) #周囲情報と方向を与える
  # 壁がなければ指定方向へ進む
  case direction
  when WALK_UP
    if info[1] == TYPE_BLOCK
      return false
    end
    return self.walkUp
  when WALK_LEFT
    if info[3] == TYPE_BLOCK
      return false
    end
    return self.walkLeft
  when WALK_RIGHT
    if info[5] == TYPE_BLOCK
      return false
    end
    return self.walkRight
  when WALK_DOWN
    if info[7] == TYPE_BLOCK
      return false
    end
    return self.walkDown
  end
end

~~~
