- ```java
  import java.util.Arrays;
  import java.util.concurrent.ForkJoinPool;
  
  public class Main {
      public static void main(String[] args) {
          Comparable[] quickSortArray = {5, 3, 8, 4, 2, 7, 1, 10, 6, 9};  // Example data
          Comparable[] mergeSortArray = {5, 3, 8, 4, 2, 7, 1, 10, 6, 9};  // Example data
  
          // QuickSort Example
          ForkJoinPool pool = new ForkJoinPool();
          QuickSortTask quickSortTask = new QuickSortTask(quickSortArray, 0, quickSortArray.length - 1);
          pool.invoke(quickSortTask);
  
          // MergeSort Example
          MergeSortTask mergeSortTask = new MergeSortTask(mergeSortArray, 0, mergeSortArray.length - 1);
          pool.invoke(mergeSortTask);
  
          // Print the sorted arrays
          System.out.println("QuickSorted array: " + Arrays.toString(quickSortArray));
          System.out.println("MergeSorted array: " + Arrays.toString(mergeSortArray));
      }
  }
  ///////////////////////////////////////////////////////
  import java.util.concurrent.RecursiveAction;
  
  public class MergeSortTask extends RecursiveAction {
      private static final int THRESHOLD = 100;
      private Comparable[] array;
      private int left;
      private int right;
  
      public MergeSortTask(Comparable[] array, int left, int right) {
          this.array = array;
          this.left = left;
          this.right = right;
      }
  
      @Override
      protected void compute() {
          if (right - left < THRESHOLD) {
              insertionSort(array, left, right);
          } else {
              int mid = (left + right) / 2;
              MergeSortTask leftTask = new MergeSortTask(array, left, mid);
              MergeSortTask rightTask = new MergeSortTask(array, mid + 1, right);
              invokeAll(leftTask, rightTask);
              merge(array, left, mid, right);
          }
      }
  
      private void merge(Comparable[] array, int left, int mid, int right) {
          Comparable[] temp = new Comparable[right - left + 1];
          int i = left, j = mid + 1, k = 0;
  
          while (i <= mid && j <= right) {
              if (array[i].compareTo(array[j]) <= 0) {
                  temp[k++] = array[i++];
              } else {
                  temp[k++] = array[j++];
              }
          }
  
          while (i <= mid) {
              temp[k++] = array[i++];
          }
  
          while (j <= right) {
              temp[k++] = array[j++];
          }
  
          System.arraycopy(temp, 0, array, left, temp.length);
      }
  
      private void insertionSort(Comparable[] array, int left, int right) {
          for (int i = left + 1; i <= right; i++) {
              Comparable key = array[i];
              int j = i - 1;
              while (j >= left && array[j].compareTo(key) > 0) {
                  array[j + 1] = array[j];
                  j--;
              }
              array[j + 1] = key;
          }
      }
  }
  /////////////////////////////////////////////////////////////////////
  import java.util.concurrent.RecursiveAction;
  
  public class QuickSortTask extends RecursiveAction {
      private static final int THRESHOLD = 100;
      private Comparable[] array;
      private int left;
      private int right;
  
      public QuickSortTask(Comparable[] array, int left, int right) {
          this.array = array;
          this.left = left;
          this.right = right;
      }
  
      @Override
      protected void compute() {
          if (right - left < THRESHOLD) {
              insertionSort(array, left, right);
          } else {
              int pivotIndex = partition(array, left, right);
              QuickSortTask leftTask = new QuickSortTask(array, left, pivotIndex - 1);
              QuickSortTask rightTask = new QuickSortTask(array, pivotIndex + 1, right);
              invokeAll(leftTask, rightTask);
          }
      }
  
      private int partition(Comparable[] array, int left, int right) {
          Comparable pivot = array[right];
          int i = left - 1;
          for (int j = left; j < right; j++) {
              if (array[j].compareTo(pivot) < 0) {
                  i++;
                  swap(array, i, j);
              }
          }
          swap(array, i + 1, right);
          return i + 1;
      }
  
      private void swap(Comparable[] array, int i, int j) {
          Comparable temp = array[i];
          array[i] = array[j];
          array[j] = temp;
      }
  
      private void insertionSort(Comparable[] array, int left, int right) {
          for (int i = left + 1; i <= right; i++) {
              Comparable key = array[i];
              int j = i - 1;
              while (j >= left && array[j].compareTo(key) > 0) {
                  array[j + 1] = array[j];
                  j--;
              }
              array[j + 1] = key;
          }
      }
  }
  
  ```