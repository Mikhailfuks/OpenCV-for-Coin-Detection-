#include <opencv2/opencv.hpp>
#include <iostream>

using namespace cv;
using namespace std;

void detectCoins(const Mat& input) {
    Mat gray, blurred, edges, masked;
    
    // Convert to grayscale
    cvtColor(input, gray, COLOR_BGR2GRAY);

    // Apply Gaussian blur to reduce noise and improve edge detection
    GaussianBlur(gray, blurred, Size(9, 9), 2);

    // Use Canny edge detector
    Canny(blurred, edges, 30, 150);

    // Find contours in the edges
    vector<vector<Point>> contours;
    findContours(edges, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);

    // Loop through each contour and find circular shapes
    for (const auto& contour : contours) {
        // Approximate the contour to a circle
        double area = contourArea(contour);
        if (area < 100) continue; // Skip small contours

        // Get the moments to find the center point
        Moments M = moments(contour);
        Point center(M.m10 / M.m00, M.m01 / M.m00);

        // Fit a circle to the contour
        float radius;
        Point2f centerPoint;
        minEnclosingCircle(contour, centerPoint, radius);

        // Optionally filter based on radius range for coins
        if(radius > 10 && radius < 100) {
            // Draw detected circles
            circle(input, centerPoint, static_cast<int>(radius), Scalar(0, 255, 0), 2);
            circle(input, center, 5, Scalar(255, 0, 0), -1); // center point
        }
    }

    // Show results
    imshow("Coin Detection", input);
}

int main(int argc, char** argv) {
    if (argc != 2) {
        cout << "Usage: " << argv[0] << " <image_path>" << endl;
        return -1;
    }

    // Load the image
    Mat image = imread(argv[1]);
    if(image.empty()) {
        cout << "Error loading image!" << endl;
        return -1;
    }

    // Perform coin detection
    detectCoins(image);

    // Wait until a key is pressed
    waitKey(0);
    return 0;
}
