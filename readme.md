LinqExpressionSnippets
====
`System.Linq.Expression` 네임스페이스 아래의 기능들에 대한 복붙용 코드 조각들.<br>

Lambda를 Expression으로 전달받기
----

__람다의 타입 판별__
```cs
public static void Foo<T>(Expression<Func<T>> f)
        {
            // Foo(() => Math.Abs(1));
            if (f.Body is MethodCallExpression)
                Console.WriteLine("MethodCallExpression");

            // Foo(() => pos.x);
            if (f.Body is MemberExpression)
                Console.WriteLine("MemberExpression");

            // Foo(() => 10);
            if (f.Body is ConstantExpression)
                Console.WriteLine("ConstExpression");

            // Foo(() => a > b);
            if (f.Body is BinaryExpression)
                Console.WriteLine("BinaryExpression");
        }
```

__MethodCallExpression__
```cs
// Foo(() => Math.Abs(1));
            if (f.Body is MethodCallExpression)
            {
                var body = f.Body as MethodCallExpression;

                // System.Math
                Console.WriteLine(body.Method.DeclaringType);

                // Abs
                Console.WriteLine(body.Method.Name);

                // 1
                Console.WriteLine(body.Arguments[0]);
            }
```

__MemberExpression__
```cs
// Foo(() => pos.x);
            if (f.Body is MemberExpression)
            {
                var body = f.Body as MemberExpression;

                // Vector2
                Console.WriteLine(body.Member.DeclaringType);

                var fieldExp = (MemberExpression)body.Expression;
                // pos
                Console.WriteLine(fieldExp.Member.Name);

                // System.Int32
                Console.WriteLine(body.Type);

                // x
                Console.WriteLine(body.Member.Name);
            }
```    

__ConstantExpression__
```cs
// Foo(() => 10);
            if (f.Body is ConstantExpression)
            {
                var body = f.Body as ConstantExpression;

                // System.Int32
                Console.WriteLine(body.Type);

                // 10
                Console.WriteLine(body.Value);
            }
```

__BinaryExpression__
```cs
// Foo(() => a > b);
            if (f.Body is BinaryExpression)
            {
                var body = f.Body as BinaryExpression;

                // Expression
                Console.WriteLine(body.Left);
                // Expression
                Console.WriteLine(body.Right);

                // ExpressionType.GreaterThan
                Console.WriteLine(body.NodeType);
            }
```

Exression 트리 생성하기
----

__Hello World__
```cs
            var exp = Expression.Block(
                Expression.Call(
                    null,
                    typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
                    Expression.Constant("Hello World")
                ));

            var lambda = Expression.Lambda<Action>(exp);
            lambda.Compile()();
```

__local variables__
```cs
            var loc1 = Expression.Variable(typeof(int));
            var loc2 = Expression.Variable(typeof(int));

            var exp = Expression.Block(
                // 지역 변수 선언
                new ParameterExpression[] {
                    loc1, loc2
                },

                // 메소드 바디
                Expression.Assign(
                    loc1,
                    Expression.Constant(10)),
                Expression.Call(
                    null,
                    typeof(Console).GetMethod("WriteLine", new Type[] { typeof(int) }),
                    loc1
                ));

            var lambda = Expression.Lambda<Action>(exp);
            lambda.Compile()();
```

__If ~ Else__
```cs
var loc1 = Expression.Variable(typeof(int));
            var loc2 = Expression.Variable(typeof(int));

            var exp = Expression.Block(
                // 지역 변수 선언
                new ParameterExpression[] {
                    loc1, loc2
                },

                // 메소드 바디
                Expression.Assign(
                    loc1,
                    Expression.Constant(11)),

                Expression.IfThenElse(
                    Expression.Equal(loc1, Expression.Constant(10)),

                    Expression.Call(
                        null,
                        typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
                        Expression.Constant("loc1 is 10")),

                    Expression.Call(
                        null,
                        typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
                        Expression.Constant("loc1 is not 10"))
                ));

            var lambda = Expression.Lambda<Action>(exp);
            lambda.Compile()();
```

트리 끼워넣기(Injection)
----

__Before, After__
```cs
static void Zoo(Expression<Action> f)
        {
            var exp = Expression.Block(
                Expression.Call(
                    null,
                    typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
                    Expression.Constant("before `f`")),

                f.Body,

                Expression.Call(
                    null,
                    typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
                    Expression.Constant("after `f`"))
                );

            var lambda = Expression.Lambda<Action>(exp);
            lambda.Compile()();
        }
```

__트리 순회하기__
```cs
        static void Rini(Expression f)
        {
            Console.WriteLine(f);

            if (f is MethodCallExpression)
            {
                var body = f as MethodCallExpression;

                foreach (var arg in body.Arguments)
                    Rini(arg);
            }
            if (f is BinaryExpression)
            {
                var body = f as BinaryExpression;

                Rini(body.Left);
                Rini(body.Right);
            }
        }
        static void Rini(Expression<Action> f)
        {
            Rini(f.Body);
        }
```
`Expression`은 순회와, 수정에 적합한 구조는 아닙ㄴㅣ디ㅏ.