<template>
  <div>
    <div id="youtube-player"></div>
    <p>Trạng thái: {{ reached50Percent ? 'Đã xem hơn 50%' : 'Đang xem...' }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

// Props
const props = defineProps({
  videoId: {
    type: String,
    required: true // ID video YouTube, ví dụ: 'dQw4w9WgXcQ'
  }
});

// State
const player = ref(null);
const reached50Percent = ref(false);
const checkTimeInterval = ref(null);

// Khởi tạo trình phát
const initializePlayer = () => {
  player.value = new window.YT.Player('youtube-player', {
    height: '315',
    width: '560',
    videoId: props.videoId,
    playerVars: {
      enablejsapi: 1 // Kích hoạt JS API
    },
    events: {
      onReady: onPlayerReady,
      onStateChange: onPlayerStateChange
    }
  });
};

// Khi trình phát sẵn sàng
const onPlayerReady = (event) => {
  console.log('Player is ready');
  // Có thể thêm hành động khi trình phát sẵn sàng
};

// Khi trạng thái trình phát thay đổi
const onPlayerStateChange = (event) => {
  console.log(event)
  if (event.data === window.YT.PlayerState.PLAYING) {
    // Kiểm tra tiến trình khi video đang phát
    checkTimeInterval.value = setInterval(checkVideoProgress, 1000);
  } else {
    // Dừng kiểm tra khi video không phát
    if (checkTimeInterval.value) {
      clearInterval(checkTimeInterval.value);
    }
  }
};

// Kiểm tra tiến trình video
const checkVideoProgress = () => {
  if (!player.value || !player.value.getCurrentTime || !player.value.getDuration) {
    console.error('Player not initialized or missing methods');
    clearInterval(checkTimeInterval.value);
    console.log('stopped')
    return;
  }
  console.log('stopped 2')

  if (!reached50Percent.value && player.value && typeof player.value.getCurrentTime === 'function') {
    const currentTime = player.value.getCurrentTime(); // Thời gian hiện tại (giây)
    const duration = player.value.getDuration(); // Thời lượng tổng (giây)
    const percentageWatched = (currentTime / duration) * 100;
  console.log(percentageWatched)
    if (percentageWatched >= 50) {
      console.log('Người dùng đã xem hơn 50% thời lượng video!');
      reached50Percent.value = true; // Cập nhật trạng thái
      clearInterval(checkTimeInterval.value); // Dừng kiểm tra
      // Gọi hàm gửi dữ liệu đến server nếu cần
      // sendProgressToServer();
    }
  }
};

// Gửi dữ liệu đến server (ví dụ)
const sendProgressToServer = () => {
  fetch('/api/log-video-progress', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      videoId: props.videoId,
      reached50Percent: true
    })
  })
    .then(response => console.log('Logged to server:', response))
    .catch(error => console.error('Error logging progress:', error));
};

// Tải YouTube IFrame API và khởi tạo trình phát
onMounted(() => {
    initializePlayer();
});

// Dọn dẹp khi component bị hủy
onUnmounted(() => {
  if (checkTimeInterval.value) {
    clearInterval(checkTimeInterval.value);
  }
  if (player.value && typeof player.value.destroy === 'function') {
    player.value.destroy();
  }
});
</script>

<style scoped>
#youtube-player {
  margin-bottom: 20px;
}
.status {
  font-size: 18px;
  color: green;
}
</style>