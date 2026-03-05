package main

import (
	"net/http"
	"strconv"

	"github.com/gin-contrib/cors"
	"github.com/gin-gonic/gin"
)

type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

var users = []User{
	{ID: 1, Name: "Sakthivel"},
	{ID: 2, Name: "Mohan"},
}

func main() {

	router := gin.Default()

	router.Use(cors.Default())

	router.GET("/users", func(c *gin.Context) {
		c.JSON(http.StatusOK, users)
	})

	router.GET("/users/:id", func(c *gin.Context) {

		id := c.Param("id")

		for _, user := range users {
			if strconv.Itoa(user.ID) == id {
				c.JSON(http.StatusOK, user)
				return
			}
		}

		c.JSON(http.StatusNotFound, gin.H{
			"message": "User not found",
		})
	})

	router.POST("/users", func(c *gin.Context) {

		var newUser User

		if err := c.BindJSON(&newUser); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": "Invalid JSON",
			})
			return
		}

		users = append(users, newUser)

		c.JSON(http.StatusCreated, newUser)
	})

	router.PUT("/users/:id", func(c *gin.Context) {

		id := c.Param("id")
		var updatedUser User

		if err := c.BindJSON(&updatedUser); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": "Invalid JSON",
			})
			return
		}

		for i, user := range users {
			if strconv.Itoa(user.ID) == id {
				users[i].Name = updatedUser.Name
				c.JSON(http.StatusOK, users[i])
				return
			}
		}

		c.JSON(http.StatusNotFound, gin.H{
			"message": "User not found",
		})
	})

	router.DELETE("/users/:id", func(c *gin.Context) {

		id := c.Param("id")

		for i, user := range users {
			if strconv.Itoa(user.ID) == id {

				users = append(users[:i], users[i+1:]...)

				c.JSON(http.StatusOK, gin.H{
					"message": "User deleted",
				})

				return
			}
		}

		c.JSON(http.StatusNotFound, gin.H{
			"message": "User not found",
		})
	})

	router.Run(":8080")
}
