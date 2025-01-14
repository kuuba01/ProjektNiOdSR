#!/usr/bin/env python3
import rclpy  # Python Client Library for ROS 2
from rclpy.node import Node  # Handles the creation of nodes
from sensor_msgs.msg import Image  # Image is the message type
from geometry_msgs.msg import Point, Twist  # Typ wiadomości Point (do publikowania punktu) oraz Twist (do publikowania prędkości)
from std_msgs.msg import Int32  # Typ wiadomości Int32 (do publikowania liczby całkowitej)
from cv_bridge import CvBridge  # ROS2 package to convert between ROS and OpenCV Images
import cv2  # Python OpenCV library
import numpy as np
import argparse  # Biblioteka do obsługi argumentów wiersza poleceń

class MinimalSubscriber(Node):
    def __init__(self, square_size):
        super().__init__('minimal_subscriber')
        self.window_name = "camera"

        # Ustawienie długości kwadratu na podstawie argumentu
        self.square_size = square_size

        # Subskrypcja obrazu (image_raw)
        self.subscription = self.create_subscription(
            Image, 'image_raw', self.listener_callback, 10
        )

        # Wydawcy na temat '/point' (punkt), '/len' (długość kwadratu) oraz '/cmd_vel' (prędkość robota)
        self.publisher_point = self.create_publisher(Point, '/point', 10)
        self.publisher_len = self.create_publisher(Int32, '/len', 10)
        self.publisher_cmd_vel = self.create_publisher(Twist, '/cmd_vel', 10)

        # Inicjalizacja CvBridge do konwersji obrazu
        self.bridge = CvBridge()
        self.point = None

        # Timer do cyklicznego publikowania prędkości
        self.timer = self.create_timer(0.1, self.publish_velocity)  # Timer co 0.1 sekundy
        self.angular_speed = 0.5  # Ustawienie prędkości obrotu robota
        self.linear_speed = 0.0  # Prędkość liniowa początkowo ustawiona na 0

        # Wysokość i szerokość obrazu (zadaną w poprzednich wymaganiach)
        self.image_height = 512
        self.image_width = 512

    def listener_callback(self, image_data):
        # Konwersja obrazu z formatu ROS na format OpenCV
        try:
            cv_image = self.bridge.imgmsg_to_cv2(image_data, desired_encoding='bgr8')

            # Rysowanie kwadratu na obrazie, jeśli punkt został ustawiony
            if self.point is not None:
                cv2.rectangle(cv_image, self.point,
                              (self.point[0] + self.square_size, self.point[1] + self.square_size), (0, 255, 0), 3)

            # Wyświetlanie obrazu
            cv2.imshow(self.window_name, cv_image)
            cv2.waitKey(25)

            # Ustawienie callbacku dla kliknięć myszy
            cv2.setMouseCallback(self.window_name, self.draw_rectangle)
        except Exception as e:
            self.get_logger().error(f"Error in callback: {e}")

    def draw_rectangle(self, event, x, y, flags, param):
        # Reakcja na kliknięcie lewym przyciskiem myszy
        if event == cv2.EVENT_LBUTTONDOWN:
            # Ustawienie punktu (lewy górny róg kwadratu)
            self.point = (x, y)
            self.get_logger().info(f"Rectangle will start at {self.point}")

            # Publikowanie punktu na temat '/point'
            point_msg = Point()
            point_msg.x = float(x)  # Konwersja na float
            point_msg.y = float(y)  # Konwersja na float

            self.publisher_point.publish(point_msg)
            self.get_logger().info(f"Published point: ({point_msg.x}, {point_msg.y})")

            # Publikowanie długości kwadratu na temat '/len'
            len_msg = Int32()
            len_msg.data = self.square_size  # Ustawienie długości boku kwadratu
            self.publisher_len.publish(len_msg)
            self.get_logger().info(f"Published square size: {len_msg.data}")

            # Określenie prędkości robota w zależności od pozycji klikniętego punktu
            self.update_velocity(x, y)

    def update_velocity(self, x, y):
        # Jeśli punkt jest powyżej środka obrazu, robot jedzie do przodu
        if y < self.image_height // 2:
            self.linear_speed = 0.5  # Prędkość liniowa do przodu
        else:
            self.linear_speed = 0.0  # Zatrzymanie robota

    def publish_velocity(self):
        # Tworzenie wiadomości typu Twist do publikowania prędkości robota
        msg = Twist()

        # Ustawienie prędkości kątowej, aby robot obracał się w kółko
        msg.angular.z = self.angular_speed

        # Ustawienie prędkości liniowej na podstawie klikniętego punktu
        msg.linear.x = self.linear_speed

        # Publikowanie wiadomości na temat '/cmd_vel'
        self.publisher_cmd_vel.publish(msg)
        self.get_logger().info(f"Publishing velocity: linear.x = {msg.linear.x}, angular.z = {msg.angular.z}")

def main(args=None):
    # Parsowanie argumentów wiersza poleceń
    parser = argparse.ArgumentParser(description="Camera subscriber with square size.")
    parser.add_argument('-p', '--square_size', type=int, default=100, help="Size of the square (default: 100)")
    parsed_args = parser.parse_args()

    # Inicjalizacja ROS 2
    rclpy.init(args=args)

    # Przekazanie parametru square_size do węzła
    minimal_subscriber = MinimalSubscriber(square_size=parsed_args.square_size)

    rclpy.spin(minimal_subscriber)

    # Zatrzymanie węzła
    minimal_subscriber.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
