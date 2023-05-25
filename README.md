# mongodbfinal
this database for final project from DBMS2 course 

## team 8Q1S
## Members
- Bakaev Eldos 210103094@stu.sdu.edu.kz
- Mukangali Zhanibek 210103153@stu.sdu.edu.kz
- Assan Nurislam 210103229@stu.sdu.edu.kz
- Aitkazin Anuar 210103136@stu.sdu.edu.kz
- Bolatbek Sagynysh 210107092@stu.sdu.edu.kz

### This is an online cloth shop project developed using MongoDB as the database management system. The project involves several collections to manage different aspects of the shop, including Cards, Reviews, items, customers, basket, address, Shipping, and order.

# Queries
1)Cумма покупок на каждого покупателя
db.Order.aggregate([
  {
    $group: {
      _id: "$c_id",
      total_revenue: { $sum: "$amount" }
    }
  },
  {
    $lookup: {
      from: "customers",
      localField: "_id",
      foreignField: "c_id",
      as: "customer"
    }
  },
  {
    $unwind: "$customer"
  },
  {
    $project: {
      _id: "$customer.c_id",
      first_name: "$customer.first_name",
      last_name: "$customer.last_name",
      total_revenue: 1
    }
  }
])

2) топ продаваемых типов вещей
db.Items.aggregate([
  {
    $lookup: {
      from: "Basket",
      localField: "it_id",
      foreignField: "it_id",
      as: "sales"
    }
  },
  {
    $unwind: "$sales"
  },
  {
    $group: {
      _id: "$type ",
      salesCount: { $sum: "$sales.count" }
    }
  },
  {
    $sort: { salesCount: -1 }
  },
  {
    $limit: 5
  }
])

3) Скидка на определенные типы товаров
db.Items.updateMany({ "type ": "Jeans" }, { $mul: { price: 0.85 } })

4)top 5 best item review
db.reviews.aggregate([
  { $group: { _id: "$it_id", averageGrade: { $avg: "$number " }, reviewCount: { $sum: 1 } } },
  { $lookup: { from: "Items", localField: "_id", foreignField: "it_id", as: "item" } },
  { $unwind: "$item" },
  { $project: { "item.it_id": 1, "item.size ": 1, "item.price ": 1, averageGrade: 1, reviewCount: 1 } },
  { $sort: { averageGrade: -1 } },
  { $limit: 5 }
])

5)top 5 worse item review
db.reviews.aggregate([
  { $group: { _id: "$it_id", averageGrade: { $avg: "$number " }, reviewCount: { $sum: 1 } } },
  { $lookup: { from: "Items", localField: "_id", foreignField: "it_id", as: "item" } },
  { $unwind: "$item" },
  { $project: { "item.it_id": 1, "item.size ": 1, "item.price ": 1, averageGrade: 1, reviewCount: 1 } },
  { $sort: { averageGrade: 1 } },
  { $limit: 5 }
])

6)TOP 5 items 
db.Basket.aggregate([
  {
    $group: {
      _id: "$it_id",
      totalSales: { $sum: "$count" }
    }
  },
  {
    $lookup: {
      from: "Items",
      localField: "_id",
      foreignField: "it_id",
      as: "item"
    }
  },
  {
    $unwind: "$item"
  },
  {
    $project: {
      "item.it_id": 1,
      "item.type ": 1,
      "item.price ": 1,
      totalSales: 1
    }
  },
  {
    $sort: { totalSales: -1 }
  },
  {
    $limit: 5
  }
])

7)количество покупателей по городам 
db.customers.aggregate([
  {
    $lookup: {
      from: "address",
      localField: "a_id",
      foreignField: "add_id",
      as: "address"
    }
  },
  {
    $unwind: "$address"
  },
  {
    $group: {
      _id: "$address.city",
      customersCount: { $sum: 1 }
    }
  },

  {
    $sort: { customersCount: -1 }
  },
  
  {
    $limit: 5
  }
])

8)средний чек на покупателя
db.Order.aggregate([
  {
    $group: {
      _id: "$customer_id",
      averageAmount: { $avg: "$amount" }
    }
  }
])

9) среднея оценка для товаров
db.reviews.aggregate([
  {
    $group: {
      _id: "$item_id",
      averageRating: { $avg: "$number " },
      reviewCount: { $sum: 1 }
    }
  }
])

10)Топ популярных вещей по городам
db.Basket.aggregate([
  {
    $lookup: {
      from: "Items",
      localField: "it_id",
      foreignField: "it_id",
      as: "item"
    }
  },
  {
    $unwind: "$item"
  },
  {
    $lookup: {
      from: "customers",
      localField: "c_id",
      foreignField: "c_id",
      as: "customer"
    }
  },
  {
    $unwind: "$customer"
  },
  {
    $lookup: {
      from: "address",
      localField: "customer.a_id",
      foreignField: "add_id",
      as: "address"
    }
  },
  {
    $unwind: "$address"
  },
  {
    $group: {
      _id: "$address.city",
      topItems: {
        $push: {
          item: "$item.type ",
          purchaseCount: "$count"
        }
      }
    }
  },
  {
    $project: {
      city: "$_id",
      topItems: {
        $slice: ["$topItems", 2]
      }
    }
  },
  {
    $sort: {
      "_id": -1
    }
  }
]);

11)top worse item
db.Basket.aggregate([
  {
    $group: {
      _id: "$it_id",
      totalSales: { $sum: "$count" }
    }
  },
  {
    $lookup: {
      from: "Items",
      localField: "_id",
      foreignField: "it_id",
      as: "item"
    }
  },
  {
    $unwind: "$item"
  },
  {
    $project: {
      "item.it_id": 1,
      "item.type ": 1,
      "item.price ": 1,
      totalSales: 1
    }
  },
  {
    $sort: { totalSales: 1 }
  },
  {
    $limit: 5
  }
])

12)top sizes 

db.Basket.aggregate([
  { $lookup: { from: "Items", localField: "it_id", foreignField: "it_id", as: "item" } },
  { $unwind: "$item" },
  { $group: { _id: "$item.size ", totalSales: { $sum: "$count" } } },
  { $sort: { totalSales: -1 } },
  { $limit: 5 }
])

13)avg amount per month
db.Order.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$date" },
        month: { $month: "$date" }
      },
      avgAmount: { $avg: "$amount" }
    }
  },
  {
    $sort: {
      "_id.year": 1,
      "_id.month": 1
    }
  }
])

14)Avg item count per customer
db.Basket.aggregate([
  {
    $group: {
      _id: "$c_id",
      avgCount: { $avg: "$count" }
    }
  }
])

15)Avg order count per customer
db.Order.aggregate([
  {
    $group: {
      _id: "$c_id",
      avgOrders: { $avg: 1 }
    }
  }
])


