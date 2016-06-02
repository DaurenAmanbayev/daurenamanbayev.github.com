---
layout: post
title: Библиотека калькулятора на C#
categories: Library
---

Разработка калькулятора одна из самых популярных задач задаваемых студентам. Ниже моя реализация классов, для разработки полноценного калькулятора.

Описано 5 классов: Memory, ValueConverter, Compute, RPN, NotationConverter. Для большей простоты используем следующие перечисления CalcType, CalcMode, DataType, ByteMode, DegreeMode.

<pre class="cscode"><code><span class="key">&nbsp;</span></code></pre>
<pre class="cscode"><code><span class="key">using</span> System;
<span class="key">using</span> System.Collections.Generic;
<span class="key">using</span> System.Linq;
<span class="key">using</span> System.Text;
<span class="key">using</span> System.Threading.Tasks;
<span class="key">using</span> System.Text.RegularExpressions;

<span class="key">namespace</span> Calculation
{
   
    <span class="com">//--------------------------------------------</span>
    <span class="com">//MODE</span>
    <span class="key">enum</span> CalcType { Memory, Unary, Binary, Converter}   
    <span class="key">enum</span> DataType { Binary = 2, Octet = 8, Decimal = 10, Hex = 16 }
    <span class="key">enum</span> ByteMode { Byte = 1, TByte = 2, FByte = 4, EByte = 8 }
    <span class="key">enum</span> CalcMode { Simple, Engineer, Developer}
    <span class="key">enum</span> DegreeMode { Degree, Radian, Grade}
    <span class="com">//---------------------------------------------</span>
    <span class="com">//MEMORY</span>
    <span class="key">class</span> Memory
    {
        <span class="key">bool</span> buffer_used = <span class="key">false</span>;
        <span class="key">double</span> buffer = 0;
        <span class="key">public</span> Memory() { }
        <span class="key">public</span> <span class="key">void</span> MemoryOperation(<span class="key">double</span> input, <span class="key">string</span> operation)
        {
            <span class="com">/* mc - clear buffer, 
             * mr - output buffer, 
             * ms - write to buffer,
             * m+ - plus to buffer,
             * m- - minus from buffer*/</span>
            buffer_used = <span class="key">false</span>;
            <span class="key">switch</span> (operation)
            {
                <span class="key">case</span> <span class="str">"mc"</span>: buffer = 0; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"mr"</span>:
                    buffer_used = <span class="key">true</span>;
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"ms"</span>: buffer = input; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"m+"</span>: buffer += input; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"m-"</span>: buffer -= input; <span class="key">break</span>;
            }           
        }
        <span class="key">public</span> <span class="key">double</span> GetBuffer
        {
            <span class="key">get</span>
            {              
                <span class="key">return</span> buffer;
            }
        }
        <span class="key">public</span> <span class="key">bool</span> BufferUsed
        {
            <span class="key">get</span> { <span class="key">return</span> buffer_used; }
        }
    }
    <span class="com">//Value Converter</span>
    <span class="com">//USE FOR CONVERT INPUT VALUE WITH CALCULATION </span>
    <span class="com">//Examle: 5^2pi | asin(90) and other</span>
    <span class="key">class</span> ValueConverter
    {
        <span class="key">public</span> ValueConverter() { }
        <span class="com">//engineer and simple mode converter</span>
        <span class="key">public</span> <span class="key">double</span> ValueConvert(<span class="key">string</span> operation, <span class="key">double</span> left, <span class="key">double</span> right=0)
        {
            <span class="key">double</span> data;
            <span class="key">switch</span> (operation)
            {
                <span class="com">//simple mode converter</span>
                <span class="key">case</span> <span class="str">"+/-"</span>: data = -left; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"sqrt"</span>: data = Math.Sqrt(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"1/x"</span>: 
                    <span class="key">if</span> (left != 0) { data = 1 / left; }
                    <span class="key">else</span> data = 0; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"%"</span>: 
                    <span class="key">if</span> (left != 0)
                    {
                        data = (right / 100) * left;
                    }
                    <span class="key">else</span> data = 0; <span class="key">break</span>;
                <span class="com">//engineer mode converter</span>
                <span class="key">case</span> <span class="str">"sin"</span>: data = Math.Sin(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"cos"</span>: data = Math.Cos(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"tan"</span>: data = Math.Tan(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"sinh"</span>: data = Math.Sinh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"cosh"</span>: data = Math.Cosh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"tanh"</span>: data = Math.Tanh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"pi"</span>: data = Math.PI; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"asin"</span>: data = Math.Asin(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"acos"</span>: data = Math.Acos(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"atan"</span>: data = Math.Atan(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"asinh"</span>: data = Asinh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"acosh"</span>: data = Acosh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"atanh"</span>: data = Atanh(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"x^2"</span>: data = Math.Pow(left, 2); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"x^3"</span>: data = Math.Pow(left, 3); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"n!"</span>: data = factorial(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"10^x"</span>: data = Math.Pow(10, left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"ln"</span>: data = Math.Log10(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"int"</span>: data = Int(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"dms"</span>: data = dms(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"degree"</span>: data = degree(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"2pi"</span>: data = 2 * Math.PI; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"e^x"</span>: data = Math.Exp(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"frac"</span>: data = Frac(left); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"sqx^3"</span>: data = NSqrt(left, 3); <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"log"</span>: data = Math.Log(left, 10); <span class="key">break</span>;
                <span class="key">default</span>: data = 0; <span class="key">break</span>;
            }
            <span class="key">return</span> data;
        }       
        <span class="com">//developer mode converter</span>
        <span class="key">public</span> <span class="key">int</span> ValueConvert(<span class="key">string</span> operation, <span class="key">int</span> left)
        {
            <span class="key">int</span> data = 0;
            <span class="key">if</span> (operation == <span class="str">"not"</span>) <span class="key">return</span> ~left;
            <span class="key">else</span> <span class="key">return</span> data;
        }
        #region Functions
        <span class="com">//-----------------------------------</span>
        <span class="com">//inverse hyperbolic functions</span>
        <span class="key">double</span> Asinh(<span class="key">double</span> input)
        {
            <span class="key">double</span> result = 0;
            <span class="key">double</span> x = input;
            result = Math.Log10(x + Math.Sqrt(Math.Pow(x, 2) + 1));
            <span class="key">return</span> result;
        }
        <span class="key">double</span> Acosh(<span class="key">double</span> input)
        {
            <span class="key">double</span> result = 0;
            <span class="key">double</span> x = input;
            <span class="key">if</span> (x &gt;= 1) result = Math.Log10(x + Math.Sqrt(Math.Pow(x, 2) - 1));
            <span class="key">return</span> result;
        }
        <span class="key">double</span> Atanh(<span class="key">double</span> input)
        {
            <span class="key">double</span> result = 0;
            <span class="key">double</span> x = input;
            <span class="key">if</span> (-1 &lt; x &amp;&amp; x &lt; 1) result = (0.5) * Math.Log10((1 + x) / (1 - x));
            <span class="key">return</span> result;
        }
        <span class="com">//----------------------------------</span>
        <span class="key">double</span> factorial(<span class="key">double</span> n)
        {
            <span class="key">if</span> (n &gt; 1)
                <span class="key">return</span> n * factorial(n - 1);
            <span class="key">else</span>
                <span class="key">return</span> 1;
        }
        <span class="com">//converter for degree</span>
        <span class="key">public</span> <span class="key">string</span> FE(<span class="key">double</span> input)
        {
            <span class="com">//5250 ~ 5E+3 example</span>
            <span class="key">string</span> result = <span class="str">""</span>;
            <span class="key">int</span> pow = 0;
            <span class="key">double</span> degree = 10;
            <span class="key">double</span> diff = 0;
            <span class="key">while</span> (input &lt; degree)
            {
                degree = Math.Pow(degree, pow);
                pow++;
            }
            diff = degree / Math.Pow(degree, --pow);
            result = diff.ToString();
            <span class="key">if</span> (input &gt; 0) result = result + <span class="str">"E+"</span> + pow.ToString();
            <span class="key">else</span> result = result + <span class="str">"E-"</span> + pow.ToString();
            <span class="key">return</span> result;
        }
        <span class="key">double</span> Int(<span class="key">double</span> input)
        {
            <span class="key">int</span> temp = Convert.ToInt32(input);
            <span class="key">return</span> (<span class="key">double</span>)temp;
        }
        <span class="com">//extract frac 5.4951 ~ 5, 6.9 ~ 6</span>
        <span class="key">double</span> Frac(<span class="key">double</span> input)
        {
            <span class="key">return</span> input - Math.Floor(input);
        }
        <span class="com">//--------------------------------</span>
        <span class="com">//GCD great common divisor</span>
        <span class="key">double</span> gcd(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="com">//convert to int </span>
            <span class="key">double</span> a = Int(input1);
            <span class="key">double</span> b = Int(input2);
            <span class="key">if</span> (b == 0)
                <span class="key">return</span> a;
            <span class="key">else</span>
                <span class="key">return</span> gcd(b, a % b);
        }
        <span class="com">//LCM</span>
        <span class="key">double</span> lcm(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="key">double</span> a = Int(input1);
            <span class="key">double</span> b = Int(input2);
            <span class="key">return</span> (a / gcd(a, b)) * b;
        }
        <span class="com">//-------------------------------------------</span>
        <span class="key">double</span> Exp(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="key">double</span> degree = Int(input2);
            <span class="key">return</span> input1 * Math.Pow(10, degree);
        }
        <span class="key">double</span> mod(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="key">double</span> result = 0;
            <span class="key">if</span> (input2 != 0) result = input1 % input2;
            <span class="key">return</span> Int(result);
        }
        <span class="com">//NSQRT</span>
        <span class="key">double</span> NSqrt(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="key">if</span> (input2 == 0) <span class="key">return</span> 1;
            <span class="key">else</span> <span class="key">return</span> Math.Pow(input1, 1 / input2);
        }
        <span class="com">//--------------------------------------------</span>
        <span class="com">//DegreeToRadian</span>
        <span class="key">double</span> DegreeToRadian(<span class="key">double</span> angle)
        {
            <span class="key">return</span> Math.PI * angle / 180.0;
        }
        <span class="com">//RadianToDegree</span>
        <span class="key">double</span> RadianToDegree(<span class="key">double</span> angle)
        {
            <span class="key">return</span> angle * (180.0 / Math.PI);
        }
        <span class="com">//--------------------------------------------</span>
        <span class="com">//degree to dms</span>
        <span class="key">double</span> dms(<span class="key">double</span> input)
        {
            <span class="key">double</span> dd = input;
            <span class="key">double</span> deg = Math.Floor(dd);
            dd -= deg;
            dd *= 60;
            <span class="key">double</span> min = Math.Floor(dd);
            dd -= min;
            dd *= 60;
            <span class="key">double</span> sec = Math.Round(dd);
            <span class="key">return</span> deg + 0.01 * min + 0.0001 * sec;
        }
        <span class="com">//dms to degree</span>
        <span class="key">double</span> degree(<span class="key">double</span> input)
        {
            <span class="key">double</span> sec = (<span class="key">int</span>)Math.Round(input * 3600);
            <span class="key">double</span> degree = sec / 3600;
            sec = Math.Abs(sec % 3600);
            <span class="key">double</span> min = sec / 60;
            sec %= 60;
            <span class="key">return</span> degree + (min / 60) + (sec / 3600);
        }
        #endregion
    }
    <span class="com">//RegularExpressions</span>
    <span class="com">//for more effective calculations using regular expressions to parse notation and computing result</span>
    <span class="com">//1) you have to set your computing mode for Simple or Developer</span>
    <span class="com">//2) use Computing method for parsing and calculation your string </span>
    <span class="key">class</span> Compute
    {
        <span class="key">int</span> byteMode = 8;
        <span class="key">bool</span> developer = <span class="key">false</span>;
        <span class="key">bool</span> outRange = <span class="key">false</span>;
        <span class="key">public</span> Calc() { }
        <span class="key">public</span> <span class="key">void</span> isDeveloper(<span class="key">bool</span> mode)
        {
            developer = mode;
        }
        <span class="key">public</span> <span class="key">void</span> ByteMode(ByteMode mode)
        {
            byteMode = (<span class="key">int</span>)mode;
        }
        <span class="com">//----------------------------------------------------------</span>
        <span class="com">//simple calculation</span>
        <span class="key">double</span> Calculate(<span class="key">double</span> left, <span class="key">double</span> right, <span class="key">string</span> op)
        {
            <span class="key">double</span> result;
            <span class="key">switch</span> (op)
            {
                <span class="key">case</span> <span class="str">"+"</span>:
                    result = left + right;
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"-"</span>:
                    result = left - right;
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"*"</span>:
                    result = left * right;
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"."</span>:
                    result = left + DoubleConvert(right);
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">","</span>:
                    result = left + DoubleConvert(right);
                    <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"/"</span>:
                    <span class="key">if</span> (right == 0) result = 0;
                    <span class="key">else</span> result = left / right;
                    <span class="key">break</span>;
                <span class="key">default</span>:
                    result = 0;
                    <span class="key">break</span>;
            }

            <span class="key">return</span> result;
        }
        <span class="com">//developer calculation</span>
        <span class="key">int</span> DevCalculate(<span class="key">int</span> left, <span class="key">int</span> right, <span class="key">string</span> op)
        {
            Range(left); Range(right);
            <span class="key">int</span> result = 0;
            <span class="key">switch</span> (op)
            {
                <span class="key">case</span> <span class="str">"+"</span>: result = left + right; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"-"</span>: result = left - right; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"*"</span>: result = left * right; <span class="key">break</span>;
                <span class="key">case</span> <span class="str">"/"</span>:
                    <span class="key">if</span> (right == 0) result = 0;
                    <span class="key">else</span> result = (<span class="key">int</span>)(left / right);
                    <span class="key">break</span>;
                <span class="key">default</span>: result = 0; <span class="key">break</span>;
                <span class="com">//-------------------------------------------</span>
                <span class="key">case</span> <span class="str">"&amp;"</span>: result = left &amp; right; <span class="key">break</span>;<span class="com">//Ans</span>
                <span class="key">case</span> <span class="str">"#"</span>: result = left | right; <span class="key">break</span>;<span class="com">//Or</span>
                <span class="key">case</span> <span class="str">"^"</span>: result = left ^ right; <span class="key">break</span>;<span class="com">//Xor</span>
                <span class="key">case</span> <span class="str">"&lt;"</span>: result = left &lt;&lt; right; <span class="key">break</span>;<span class="com">//Lsh</span>
                <span class="key">case</span> <span class="str">"&gt;"</span>: result = left &gt;&gt; right; <span class="key">break</span>;<span class="com">//Arsh </span>
                <span class="key">case</span> <span class="str">"?"</span>:<span class="com">//Rsh</span>
                    {
                        <span class="key">int</span> temp = (<span class="key">int</span>)left;
                        result = temp &gt;&gt; right; <span class="key">break</span>;
                    }
                <span class="key">case</span> <span class="str">":"</span>: result = RoL((Int32)left, right); <span class="key">break</span>;<span class="com">//RoL</span>
                <span class="key">case</span> <span class="str">";"</span>: result = RoR((Int32)left, right); <span class="key">break</span>;<span class="com">//RoR</span>
            }
            <span class="key">return</span> result;
        }
        <span class="key">double</span> DoubleConvert(<span class="key">double</span> input)
        {
            <span class="key">string</span> res = input.ToString();
            <span class="key">int</span> degree = res.Length;
            <span class="key">return</span> Math.Round(input * (1 / Math.Pow(10, degree)),3);
        }
        <span class="com">//-------------------------------------------------------------------------</span>
        <span class="com">//Regex simple mode computing</span>
        <span class="key">public</span> <span class="key">void</span> Computing(<span class="key">ref</span> <span class="key">string</span> calculation)
        {
            <span class="key">if</span> (developer) Developer(<span class="key">ref</span> calculation);
            <span class="key">else</span> Simple(<span class="key">ref</span> calculation);
        }
        #region Functions
        <span class="key">void</span> Simple(<span class="key">ref</span> <span class="key">string</span> calculation)
        {
            <span class="key">string</span> pattern = @<span class="str">"\s*(?&lt;number&gt;\d+)\s*(?&lt;operator&gt;(\+|\-|\*|\/|\.|\,))*\s*"</span>;
            Regex regex = <span class="key">new</span> Regex(pattern);
            MatchCollection mc = regex.Matches(calculation);

            <span class="key">string</span> op = <span class="key">string</span>.Empty;
            <span class="key">bool</span> first = <span class="key">true</span>;
            <span class="key">double</span>? result = <span class="key">null</span>;
            <span class="key">double</span> left = 0;
            <span class="key">double</span> right = 0;

            <span class="key">foreach</span> (Match m <span class="key">in</span> mc)
            {
                <span class="key">if</span> (!first)
                {
                    right = <span class="key">double</span>.Parse(m.Groups[<span class="str">"number"</span>].Value);
                    <span class="key">if</span> (result.HasValue)
                    {
                        result = Calculate(result.Value, right, op);
                    }
                    <span class="key">else</span>
                    {
                        result = Calculate(left, right, op);
                    }
                }

                left = <span class="key">double</span>.Parse(m.Groups[<span class="str">"number"</span>].Value);
                op = m.Groups[<span class="str">"operator"</span>].Value;
                first = <span class="key">false</span>;
            }
            calculation = result.ToString();
        }
        <span class="com">//Regex developer mode computing</span>
        <span class="key">void</span> Developer(<span class="key">ref</span> <span class="key">string</span> calculation)
        {
            <span class="key">string</span> pattern = @<span class="str">"\s*(?&lt;number&gt;\d+)\s*(?&lt;operator&gt;(\+|\-|\*|\/|\.|\&amp;|\#|\^|\&lt;|\&gt;|\?|\:|\;))*\s*"</span>;
            Regex regex = <span class="key">new</span> Regex(pattern);
            MatchCollection mc = regex.Matches(calculation);
            <span class="key">string</span> op = <span class="key">string</span>.Empty;
            <span class="key">bool</span> first = <span class="key">true</span>;
            <span class="key">int</span>? result = <span class="key">null</span>;
            <span class="key">int</span> left = 0;
            <span class="key">int</span> right = 0;
            <span class="key">foreach</span> (Match m <span class="key">in</span> mc)
            {
                <span class="key">if</span> (!first)
                {
                    right = <span class="key">int</span>.Parse(m.Groups[<span class="str">"number"</span>].Value);
                    <span class="key">if</span> (result.HasValue)
                    {
                        result = DevCalculate(result.Value, right, op);
                    }
                    <span class="key">else</span>
                    {
                        result = DevCalculate(left, right, op);
                    }
                }

                left = <span class="key">int</span>.Parse(m.Groups[<span class="str">"number"</span>].Value);
                op = m.Groups[<span class="str">"operator"</span>].Value;
                first = <span class="key">false</span>;
            }
            calculation = result.ToString();
        }
        <span class="com">//-------------------------------------------------------       </span>
        <span class="key">void</span> Range(<span class="key">int</span> input)
        {
            <span class="key">if</span> (!outRange)
            {
                outRange=CheckRange(input);
            }
        }
        <span class="key">bool</span> CheckRange(<span class="key">int</span> input)
        {
            <span class="key">uint</span> finrange;
            ByteRange(byteMode, <span class="key">out</span> finrange);
            <span class="key">if</span> (byteMode == 8) <span class="key">return</span> <span class="key">true</span>;
            <span class="key">else</span> <span class="key">if</span> (byteMode != 8 &amp;&amp; input &lt;= finrange) <span class="key">return</span> <span class="key">true</span>;
            <span class="key">else</span> <span class="key">return</span> <span class="key">false</span>;
        }
        <span class="key">void</span> ByteRange(<span class="key">int</span> powmode,<span class="key">out</span> <span class="key">uint</span> finrange)
        {          
            finrange = 0;
            <span class="key">int</span> findegree = powmode * 8 - 1;         
            <span class="key">int</span> startdegree = findegree - 8;               
            finrange = (<span class="key">uint</span>)Math.Pow(2, findegree) - 1;      
        }
        <span class="com">//----------------------------------------------------------</span>
        <span class="key">public</span> Int32 RoL(Int32 input, <span class="key">int</span> shift)
        {
            shift %= 31;
            <span class="key">return</span> ((input &lt;&lt; shift) | (input &gt;&gt; (32 - shift)));
        }
        <span class="key">public</span> Int32 RoR(Int32 input, <span class="key">int</span> shift)
        {
            shift %= 31;
            <span class="key">return</span> ((input &gt;&gt; shift) | (input &lt;&lt; (32 - shift)));
        }
        <span class="com">//---------------------------------------------------</span>
        <span class="key">public</span> <span class="key">double</span> Parse(<span class="key">string</span> input)
        {
            <span class="key">double</span> data = 0;
            <span class="key">if</span> (<span class="key">double</span>.TryParse(input, <span class="key">out</span> data))
            {
                <span class="key">return</span> data;
            }
            <span class="key">else</span> <span class="key">return</span> 0;
        }
        #endregion
    }
    <span class="com">//Reverse polish notation</span>
    <span class="com">//uaw this for engineer calculation mode</span>
    <span class="com">//computing parse and calculation string with priority of operands</span>
    <span class="key">class</span> RPN
    {
        <span class="key">public</span> RPN()
        {
            operators = <span class="key">new</span> List&lt;<span class="key">string</span>&gt;(standart_operators);

        }
        <span class="key">private</span> List&lt;<span class="key">string</span>&gt; operators;
        <span class="key">private</span> List&lt;<span class="key">string</span>&gt; standart_operators =
            <span class="key">new</span> List&lt;<span class="key">string</span>&gt;(<span class="key">new</span> <span class="key">string</span>[] { <span class="str">"("</span>, <span class="str">")"</span>, <span class="str">"+"</span>, <span class="str">"-"</span>, <span class="str">"*"</span>, <span class="str">"/"</span>, <span class="str">"^"</span>, <span class="str">"#"</span> });
        <span class="com">//# - sqrt(x,y)</span>
        <span class="key">private</span> IEnumerable&lt;<span class="key">string</span>&gt; Separate(<span class="key">string</span> input)
        {
            <span class="key">int</span> pos = 0;
            <span class="key">while</span> (pos &lt; input.Length)
            {
                <span class="key">string</span> s = <span class="key">string</span>.Empty + input[pos];
                <span class="key">if</span> (!standart_operators.Contains(input[pos].ToString()))
                {
                    <span class="key">if</span> (Char.IsDigit(input[pos]))
                        <span class="key">for</span> (<span class="key">int</span> i = pos + 1; i &lt; input.Length &amp;&amp;
                            (Char.IsDigit(input[i]) || input[i] == <span class="str">','</span> || input[i] == <span class="str">'.'</span>); i++)
                            s += input[i];
                    <span class="key">else</span> <span class="key">if</span> (Char.IsLetter(input[pos]))
                        <span class="key">for</span> (<span class="key">int</span> i = pos + 1; i &lt; input.Length &amp;&amp;
                            (Char.IsLetter(input[i]) || Char.IsDigit(input[i])); i++)
                            s += input[i];
                }
                <span class="key">yield</span> <span class="key">return</span> s;
                pos += s.Length;
            }
        }
        <span class="com">//operation priority</span>
        <span class="key">private</span> <span class="key">byte</span> GetPriority(<span class="key">string</span> s)
        {
            <span class="key">switch</span> (s)
            {
                <span class="key">case</span> <span class="str">"("</span>:
                <span class="key">case</span> <span class="str">")"</span>:
                    <span class="key">return</span> 0;
                <span class="key">case</span> <span class="str">"+"</span>:
                <span class="key">case</span> <span class="str">"-"</span>:
                    <span class="key">return</span> 1;
                <span class="key">case</span> <span class="str">"*"</span>:
                <span class="key">case</span> <span class="str">"/"</span>:
                    <span class="key">return</span> 2;
                <span class="key">case</span> <span class="str">"^"</span>:
                <span class="key">case</span> <span class="str">"#"</span>:
                    <span class="key">return</span> 3;
                <span class="key">default</span>:
                    <span class="key">return</span> 4;
            }
        }
        <span class="com">//RPN convert </span>
        <span class="key">string</span>[] ConvertToPostfixNotation(<span class="key">string</span> input)
        {
            List&lt;<span class="key">string</span>&gt; outputSeparated = <span class="key">new</span> List&lt;<span class="key">string</span>&gt;();
            Stack&lt;<span class="key">string</span>&gt; stack = <span class="key">new</span> Stack&lt;<span class="key">string</span>&gt;();
            <span class="key">foreach</span> (<span class="key">string</span> c <span class="key">in</span> Separate(input))
            {
                <span class="key">if</span> (operators.Contains(c))
                {
                    <span class="key">if</span> (stack.Count &gt; 0 &amp;&amp; !c.Equals(<span class="str">"("</span>))
                    {
                        <span class="key">if</span> (c.Equals(<span class="str">")"</span>))
                        {
                            <span class="key">string</span> s = stack.Pop();
                            <span class="key">while</span> (s != <span class="str">"("</span>)
                            {
                                outputSeparated.Add(s);
                                s = stack.Pop();
                            }
                        }
                        <span class="key">else</span> <span class="key">if</span> (GetPriority(c) &gt; GetPriority(stack.Peek()))
                            stack.Push(c);
                        <span class="key">else</span>
                        {
                            <span class="key">while</span> (stack.Count &gt; 0 &amp;&amp; GetPriority(c) &lt;= GetPriority(stack.Peek()))
                                outputSeparated.Add(stack.Pop());
                            stack.Push(c);
                        }
                    }
                    <span class="key">else</span>
                        stack.Push(c);
                }
                <span class="key">else</span>
                    outputSeparated.Add(c);
            }
            <span class="key">if</span> (stack.Count &gt; 0)
                <span class="key">foreach</span> (<span class="key">string</span> c <span class="key">in</span> stack)
                    outputSeparated.Add(c);

            <span class="key">return</span> outputSeparated.ToArray();
        }
        <span class="com">//return results</span>
        <span class="key">public</span> <span class="key">decimal</span> Result(<span class="key">string</span> input)
        {
            Stack&lt;<span class="key">string</span>&gt; stack = <span class="key">new</span> Stack&lt;<span class="key">string</span>&gt;();
            Queue&lt;<span class="key">string</span>&gt; queue = <span class="key">new</span> Queue&lt;<span class="key">string</span>&gt;(ConvertToPostfixNotation(input));
            <span class="key">string</span> str = queue.Dequeue();
            <span class="key">while</span> (queue.Count &gt;= 0)
            {
                <span class="key">if</span> (!operators.Contains(str))
                {
                    stack.Push(str);
                    str = queue.Dequeue();
                }
                <span class="key">else</span>
                {
                    <span class="key">decimal</span> summ = 0;
                    <span class="key">try</span>
                    {

                        <span class="key">switch</span> (str)
                        {

                            <span class="key">case</span> <span class="str">"+"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = a + b;
                                    <span class="key">break</span>;
                                }
                            <span class="key">case</span> <span class="str">"-"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = b - a;
                                    <span class="key">break</span>;
                                }
                            <span class="key">case</span> <span class="str">"*"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = b * a;
                                    <span class="key">break</span>;
                                }
                            <span class="key">case</span> <span class="str">"/"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = b / a;
                                    <span class="key">break</span>;
                                }
                            <span class="key">case</span> <span class="str">"^"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = Convert.ToDecimal(Math.Pow(Convert.ToDouble(b), Convert.ToDouble(a)));
                                    <span class="key">break</span>;
                                }
                            <span class="key">case</span> <span class="str">"#"</span>:
                                {
                                    <span class="key">decimal</span> a = Convert.ToDecimal(stack.Pop());
                                    <span class="key">decimal</span> b = Convert.ToDecimal(stack.Pop());
                                    summ = Convert.ToDecimal(NSqrt(Convert.ToDouble(b), Convert.ToDouble(a)));
                                    <span class="key">break</span>;
                                }
                        }
                    }
                    <span class="key">catch</span> (Exception ex)
                    {

                    }
                    stack.Push(summ.ToString());
                    <span class="key">if</span> (queue.Count &gt; 0)
                        str = queue.Dequeue();
                    <span class="key">else</span>
                        <span class="key">break</span>;
                }

            }
            <span class="key">return</span> Convert.ToDecimal(stack.Pop());
        }
        <span class="com">//expression syntax checking</span>
        <span class="key">public</span> <span class="key">bool</span> CheckSyntax(<span class="key">string</span> str)
        {
            <span class="key">int</span> tmpdata;
            <span class="key">string</span> operands = <span class="str">"* ()+-*/=,"</span>;
            <span class="com">//checking string is correct</span>
            <span class="key">string</span> tmp = str;
            <span class="key">for</span> (<span class="key">int</span> i = 0; i &lt; operands.Length; i++)
            {
                <span class="key">string</span> currentOperands = operands[i].ToString();
                tmp = tmp.Replace(currentOperands, <span class="str">""</span>);
            }
            <span class="key">bool</span> isNumeric = <span class="key">int</span>.TryParse(tmp, <span class="key">out</span> tmpdata);
            <span class="key">if</span> (isNumeric) <span class="key">return</span> <span class="key">true</span>;
            <span class="key">else</span> <span class="key">return</span> <span class="key">false</span>;
        }
        <span class="com">//extract error symbol</span>
        <span class="key">string</span> TrimError(<span class="key">string</span> s)
        {
            <span class="key">char</span>[] charsToTrim = { <span class="str">'*'</span>, <span class="str">' '</span>, <span class="str">'/'</span>, <span class="str">'+'</span>, <span class="str">'-'</span>, <span class="str">'.'</span>, <span class="str">','</span> };
            s = s.Trim(charsToTrim);
            <span class="key">return</span> s;
        }
        <span class="com">//NSQRT </span>
        <span class="key">double</span> NSqrt(<span class="key">double</span> input1, <span class="key">double</span> input2)
        {
            <span class="key">if</span> (input2 == 0) <span class="key">return</span> 1;
            <span class="key">else</span> <span class="key">return</span> Math.Pow(input1, 1 / input2);
        }
    }
    <span class="com">//Value notation converter</span>
    <span class="com">//2, 8, 10, 16 bit notation</span>
    <span class="key">class</span> NotationConverter
    {
        <span class="key">string</span> zero = <span class="str">"0"</span>;
        <span class="key">public</span> NotationConverter()
        { }
        <span class="com">//2 система</span>
        <span class="key">public</span> <span class="key">string</span> BinToOct(<span class="key">string</span> bin)
        {
            <span class="key">try</span>
            {
                <span class="key">int</span> dec = Convert.ToInt32(bin, 2);
                <span class="key">return</span> TenToBase(dec, 8);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> BinToHex(<span class="key">string</span> bin)
        {
            <span class="key">try</span>
            {
                <span class="key">int</span> dec = Convert.ToInt32(bin, 2);
                <span class="key">return</span> TenToBase(dec, 16);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> BinToDec(<span class="key">string</span> bin)
        {
            <span class="key">try</span>
            {
                <span class="key">return</span> Convert.ToInt32(bin, 2).ToString();
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="com">//8 bit notation</span>
        <span class="key">public</span> <span class="key">string</span> OctToHex(<span class="key">string</span> oct)
        {
            <span class="key">try</span>
            {
                <span class="key">int</span> dec = Convert.ToInt32(oct, 8);
                <span class="key">return</span> TenToBase(dec, 16);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> OctToBin(<span class="key">string</span> oct)
        {
            <span class="key">try</span>
            {
                <span class="key">int</span> dec = Convert.ToInt32(oct, 8);
                <span class="key">return</span> TenToBase(dec, 2);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="com">//16 bit notation</span>
        <span class="key">public</span> <span class="key">string</span> HexToDec(<span class="key">string</span> hex)
        {
            <span class="key">try</span>
            {
                <span class="key">return</span> (Convert.ToInt32(hex, 16)).ToString();
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> HexToOct(<span class="key">string</span> hex)
        {
            <span class="key">try</span>
            {
                <span class="key">int</span> dec = Convert.ToInt32(hex, 16);
                <span class="key">return</span> TenToBase(dec, 8);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> HexToBin(<span class="key">string</span> hex)
        {
            <span class="key">try</span>
            {
                <span class="key">return</span> Convert.ToString(Convert.ToInt32(hex, 16), 2).PadLeft(12, <span class="str">'0'</span>);
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="com">//10 bit notation</span>
        <span class="key">public</span> <span class="key">string</span> TenToBase(<span class="key">int</span> value, <span class="key">int</span> Base)
        {
            <span class="key">try</span>
            {
                <span class="key">string</span> strBin = <span class="str">""</span>;
                <span class="key">char</span>[] cHexa = <span class="key">new</span> <span class="key">char</span>[] { <span class="str">'A'</span>, <span class="str">'B'</span>, <span class="str">'C'</span>, <span class="str">'D'</span>, <span class="str">'E'</span>, <span class="str">'F'</span> };
                <span class="key">int</span>[] result = <span class="key">new</span> <span class="key">int</span>[32];
                <span class="key">int</span> MaxBit = 32;
                <span class="key">for</span> (; value &gt; 0; value /= Base)
                {
                    <span class="key">int</span> rem = value % Base;
                    result[--MaxBit] = rem;
                }
                <span class="key">for</span> (<span class="key">int</span> i = 0; i &lt; result.Length; i++)
                    <span class="key">if</span> ((<span class="key">int</span>)result.GetValue(i) &gt;= 10)
                        strBin += cHexa[(<span class="key">int</span>)result.GetValue(i) % 10];
                    <span class="key">else</span>
                        strBin += result.GetValue(i);
                strBin = strBin.TrimStart(<span class="key">new</span> <span class="key">char</span>[] { <span class="str">'0'</span> });
                <span class="key">return</span> strBin;
            }
            <span class="key">catch</span>
            {
                <span class="key">return</span> zero;
            }
        }
        <span class="key">public</span> <span class="key">string</span> BaseToTen(<span class="key">string</span> input, <span class="key">int</span> divide)
        {
            <span class="key">try</span>
            {
                <span class="key">decimal</span> dec = Convert.ToInt32(input, divide);
                <span class="key">return</span> dec.ToString();
            }
            <span class="key">catch</span> (Exception)
            {
                <span class="key">return</span> zero;
            }
        }


    }
}
</code></pre>
</div>
