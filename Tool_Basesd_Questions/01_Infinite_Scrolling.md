import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.interactions.Actions;
import java.time.Duration;

public class LazyLoader {
    public static void scrollToBottom(WebDriver driver) throws InterruptedException {
        Actions actions = new Actions(driver);
        JavascriptExecutor js = (JavascriptExecutor) driver;

        long lastHeight = (long) js.executeScript("return document.body.scrollHeight");

        while (true) {
            // 1. Scroll down by a chunk of pixels
            actions.scrollByAmount(0, 500).perform();

            // 2. Wait for AJAX / network requests to load new elements
            Thread.sleep(1000); 

            // 3. Check the new total height of the page
            long newHeight = (long) js.executeScript("return document.body.scrollHeight");

            // 4. Break the loop if the height hasn't changed (reached bottom)
            if (newHeight == lastHeight) {
                break;
            }
            lastHeight = newHeight;
        }
    }
}
