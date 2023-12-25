# 3sem_lab7
## Homework
### Задание 1
Цели работы:
Научиться работать с механизмом рефлексии языка C#.
Задание№1
Создайте проект библиотеки классов со следующей диаграммой классов
В созданном проекте реализуйте класс пользовательского атрибута со свойством Comment. Примените атрибут с произвольным комментарием к каждому классу диаграммы классов.
Создайте приложение, которое будет подключать созданную библиотеку классов и средствами рефлексии генерировать файл xml-представления всей диаграммы классов библиотеки, в том числе с созданными пользовательскими атрибутами.

```
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System;
using System.Reflection;
using AnimalLibrary;

public class Program
{
    static IEnumerable<Type> GetClasses(string nameSpace)
    {/*объявляется статический метод GetClasses, который принимает строку с именем пространства имен 
      * nameSpace и возвращает перечисление (IEnumerable) типов, находящихся в указанном пространстве имен*/
        var asm = Assembly.Load(nameSpace);
        //создается переменная asm, в которой мы вызываем метод Assembly.Load для загрузки сборки с указанным именем пространства имен.
        return asm.GetTypes()//мы вызываем метод GetTypes() для получения всех типов в этой сборке.
                  .Where(type => type.Namespace == nameSpace);
    }//метод расширения Where, чтобы отфильтровать только те типы, чье пространство имен совпадает с указанным пространством имен.

    public static void Main(string[] args)
    {//создается экземпляр класса StreamWriter для записи данных в файл MyLibrary.xml.
        using (StreamWriter fout = new StreamWriter(@"..\..\..\..\MyLibrary.xml"))//создание файла
        {//мы используем оператор using для создания объекта StreamWriter с именем fout и указываем путь к файлу "MyLibrary.xml"
            fout.WriteLine("<?xml version=\"1.0\" encoding=\"UTF-8\"?>");
            // Затем мы записываем строки <Library> и <name>ClassLibrary1</name>, чтобы начать создание XML-файла.
            fout.WriteLine("<Library>");//В файл записываются первые три строки с указанием версии XML и открывается корневой элемент Library.
            fout.WriteLine("<name>ClassLibrary1</name>");

            foreach (var myType in GetClasses("AnimalLibrary"))
            {//Внутри цикла выводится информация о типе в консоль, а также записывается в файл элемент type с информацией о типе, его модификаторах и базовом типе.
                Console.WriteLine("typename=" + myType.Name + ":");
                //- myType.Name возвращает имя типа.
                //Метод IsAbstract возвращает true, если тип является абстрактным
                Console.WriteLine("abstract=" + myType.IsAbstract + ";");
                Console.WriteLine("public=" + myType.IsPublic + ";");
                // myType.BaseType.Name возвращает имя базового типа данного типа.
                Console.WriteLine("basetype=" + myType.BaseType.Name + ";");
                fout.WriteLine("\t<type>");
                fout.WriteLine("\t\t<name>" + myType.Name + "</name>");
                //Затем мы записываем начало элемента <type> в XML-файл.
                if (myType.IsPublic || myType.IsAbstract)
                {/*Если тип является публичным или абстрактным, мы записываем начало элемента <modifiers> и 
                  * дополнительно записываем "public" или "abstract" в зависимости от того, какой модификатор присутствует*/
                    fout.Write("\t\t<modifiers>");
                    //В конце, если модификаторы присутствуют, закрываем элемент <modifiers>
                    if (myType.IsPublic) fout.Write("public ");
                    if (myType.IsAbstract) fout.Write("abstract");

                    fout.WriteLine("</modifiers>");
                }
                fout.WriteLine("\t\t<basetype>" + myType.BaseType.Name + "</basetype>");

                foreach (MemberInfo member in myType.GetMembers(BindingFlags.DeclaredOnly
                    | BindingFlags.Instance | BindingFlags.Public | BindingFlags.Static))
                {/* происходит цикл по всем членам типа (member), полученным с использованием метода GetMembers и 
                  * заданными флагами, указывающими на необходимость получения только объявленных членов, экземпляров, 
                  * публичных и статических членов.*/
                    fout.WriteLine("\t\t<member>");
                    fout.WriteLine("\t\t\t<name>" + member.Name + "</name>");
                    fout.WriteLine("\t<membertype>" + member.MemberType + "</membertype>");
                    Console.WriteLine("\tname=" + member.Name);
                    Console.WriteLine("\tmembertype=" + member.MemberType);
                    /*Внутри цикла выводится информация о каждом члене в консоль, а также записывается в файл 
                     * соответствующий элемент с информацией о его имени, типе и специфической информации для полей и свойств*/
                    if (member.MemberType == MemberTypes.Field)
                    {//проверяется, явл ли тип члена MemberType равным MemberTypes.Field. Он относится к членам, которые являются полями (variables) в классе.
                        FieldInfo field = myType.GetField(member.Name);
                        //Здесь создается переменная field типа FieldInfo, которая содержит информацию о поле с именем
                        //Здесь проверяется, была ли успешно найдена информация о поле (не равна ли field значению null)
                        if (field != null)
                        {
                            Type fieldType = field.FieldType;
                            //\t\t используется для добавления отступа (табуляции) в выводе. fieldType.Name возвращает имя типа поля.
                            Console.WriteLine("\t\ttype=" + fieldType.Name);
                            fout.WriteLine("\t\t\t<fieldtype>" + fieldType.Name + "</fieldtype>");
                        }//объект класса StreamWriter. <fieldtype> и </fieldtype> являются открывающим и закрывающим тегами для помещения информации о типе поля.
                    }

                    if (member.MemberType == MemberTypes.Property)
                    {//В этой строке проверяется, является ли тип члена MemberType равным MemberTypes.Property
                        PropertyInfo prop = myType.GetProperty(member.Name);
                        if (prop != null)
                        {
                            Type propertyType = prop.PropertyType;
                            Console.WriteLine("\t\ttype=" + propertyType.Name);
                            fout.WriteLine("\t\t\t<propertytype>" + propertyType.Name + "</propertytype>");
                        }
                    }

                    fout.WriteLine("\t\t</member>");
                }//После завершения цикла по членам типа закрывается элемент type и происходит переход к следующему типу.
                Console.WriteLine();
                fout.WriteLine("\t</type>");
            }//По окончании цикла по типам записывается закрывающий элемент Library и закрывается файл StreamWriter.
            fout.WriteLine("</Library>");
        }
    }
}

```

AnimalLibrary:

```
using System;

namespace AnimalLibrary
{
    [AttributeUsage(AttributeTargets.Class)]
    //определяет пользовательский атрибут CommentAtt и указывает, что атрибут может применяться только к классам (AttributeTargets.Class)
    public class CommentAtt : Attribute
    {
        public string Comment { get; set; }

        public CommentAtt(string comment)
        {
            Comment = comment;
        }
    }

    public enum AnimalClassification
    //определяется перечисление AnimalClassification, которое представляет классификацию животных - травоядные, хищники, всеядные
    {
        Herbivores,
        Carnivores,
        Omnivores
    }

    public enum FavouriteFood
    //определяется перечисление FavouriteFood, которое представляет любимую пищу животных - мясо, растения, все
    {
        Meat,
        Plants,
        Everything
    }

    [CommentAtt("Абстрактный класс для объектов, представляющих животных")]
    public abstract class Animal
    {
        private AnimalClassification classification;

        public string Name { get; set; }
        public string Country { get; set; }
        public string Description { get; set; }
        public bool HideFromOtherAnimals { get; set; }

        public abstract void SayHello();
        public abstract FavouriteFood GetFavouriteFood();

        public Animal(string country, string name, string description, bool hideFromOtherAnimals)
        {
            Name = name;
            Country = country;
            Description = description;
            HideFromOtherAnimals = hideFromOtherAnimals;
        }

        public void Deconstruct(out string out_name)
        {
            out_name = Name;
        }// возвращает значение свойства Name через параметр out_name

        public void Deconstruct(out string out_name, out string out_desc)
        {
            out_name = Name;
            out_desc = Description;
        }

        public void Deconstruct(out string out_name, out string out_desc, out string out_count)
        {
            out_name = Name;
            out_count = Country;
            out_desc = Description;
        }

        public void Deconstruct(out string out_name, out string out_desc, out string out_count, out bool out_hide)
        {
            out_name = Name;
            out_count = Country;
            out_desc = Description;
            out_hide = HideFromOtherAnimals;
        }
    }

    [CommentAtt("Класс описания коровы")]
    public class Cow : Animal
    {
        public Cow(string country, string name, string description, bool hideFromOtherAnimals) :
            base(country, name, description, hideFromOtherAnimals)
        {
        }

        public override FavouriteFood GetFavouriteFood()
        {
            return FavouriteFood.Plants;
        }

        public override void SayHello()
        {
            Console.WriteLine("Moo\n");
        }
    }

    [CommentAtt("Класс описания льва")]
    public class Lion : Animal
    {
        public Lion(string country, string name, string description, bool hideFromOtherAnimals) :
            base(country, name, description, hideFromOtherAnimals)
        {
        }

        public override FavouriteFood GetFavouriteFood()
        {
            return FavouriteFood.Meat;
        }

        public override void SayHello()
        {
            Console.WriteLine("Roar\n");
        }
    }

    [CommentAtt("Класс описания свиньи")]
    public class Pig : Animal
    {
        public Pig(string country, string name, string description, bool hideFromOtherAnimals) :
            base(country, name, description, hideFromOtherAnimals)
        {
        }

        public override FavouriteFood GetFavouriteFood()
        {
            return FavouriteFood.Everything;
        }

        public override void SayHello()
        {
            Console.WriteLine("Oink\n");
        }
    }
}

```
