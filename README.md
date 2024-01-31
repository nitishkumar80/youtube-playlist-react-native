## youtube-playlist-react-native



import React, { useState, useEffect } from 'react';
import { View, Text, Image, TouchableOpacity, FlatList, Modal, Dimensions } from 'react-native';
import YouTube from 'react-native-youtube-iframe';

const windowHeight = Dimensions.get('window').height;

const VideoCard = ({ video, openModal }) => {
  return (
    <TouchableOpacity onPress={() => openModal(video)} style={{ flex: 1, margin: 10, borderRadius: '25%', overflow: 'hidden', borderWidth: 1, borderColor: '#3498db', elevation: 2, shadowColor: '#000',  alignItems:'center',shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.8, shadowRadius: 2 , width:300,}}>
      <View style={{ marginBottom: 10, maxWidth: 300 }}>
        <Image source={{ uri: video.thumbnail }}   resizeMode='contain'  style={{  height: 200, marginBottom: 8 , alignItems:'center'}} />
        <Text style={{ fontSize: 18, overflow: 'hidden',  textAlign: 'center', padding: 8, flexShrink: 1 }}>{video.title}</Text>
      </View>
    </TouchableOpacity>
  );
};

const youtube-playlist = () => {
  const [playlistData, setPlaylistData] = useState([]);
  const [modalVisible, setModalVisible] = useState(false);
  const [selectedVideo, setSelectedVideo] = useState(null);
  const [nextPageToken, setNextPageToken] = useState(null);

  useEffect(() => {
    const fetchPlaylistData = async () => {
      const apiKey = 'AIzaSyDL3ZbF5ba0FyAmgmGqK5vwo0DRQtewX6U'; // Replace with your actual YouTube API key
      const playlistId = 'PLDzeHZWIZsTpukecmA2p5rhHM14bl2dHU'; // Replace with your actual YouTube playlist ID

      const apiUrl = `https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&maxResults=50&playlistId=${playlistId}&key=${apiKey}${nextPageToken ? `&pageToken=${nextPageToken}` : ''}`;

      try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        const videos = data.items.map((item) => ({
          videoId: item.snippet.resourceId.videoId,
          title: item.snippet.title,
          thumbnail: item.snippet.thumbnails.medium.url,
        }));

        if (data.nextPageToken) {
          setNextPageToken(data.nextPageToken);
        } else {
          setNextPageToken(null);
        }

        // Reset the playlist data if it's the first page
        if (!nextPageToken) {
          setPlaylistData(videos);
        } else {
          setPlaylistData((prevData) => [...prevData, ...videos]);
        }
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    if (nextPageToken || !playlistData.length) {
      fetchPlaylistData();
    }
  }, [nextPageToken, playlistData]);

  const openModal = (video) => {
    setSelectedVideo(video);
    setModalVisible(true);
  };

  const closeModal = () => {
    setSelectedVideo(null);
    setModalVisible(false);
  };

  return (
    <View style={{ flex: 1, padding: 10, }}>
      <FlatList
        data={playlistData}
        keyExtractor={(item) => item.videoId}
        renderItem={({ item }) => <VideoCard video={item} openModal={openModal} />}
        onEndReached={() => {
          if (nextPageToken) {
            fetchPlaylistData();
          }
        }}
        onEndReachedThreshold={0.1}
        ItemSeparatorComponent={() => <View style={{ height: 10 }} />}
        contentContainerStyle={{ alignItems: 'stretch' }}
        numColumns={2} // Set the number of columns
      />

      <Modal
        animationType="slide"
        transparent={false}
        visible={modalVisible}
        onRequestClose={closeModal}
      >
        <View style={{ flex: 1 }}>
          <TouchableOpacity onPress={closeModal}>
            <Text style={{ fontSize: 20, padding: 20 }}>Close</Text>
          </TouchableOpacity>
          {selectedVideo && (
            <YouTube
              videoId={selectedVideo.videoId}
              play={true}
              fullscreen={true}
              loop={false}
              apiKey="AIzaSyDL3ZbF5ba0FyAmgmGqK5vwo0DRQtewX6U" // Replace with your actual YouTube API key
              style={{ flex: 1, height: windowHeight }}
            />
          )}
        </View>
      </Modal>
    </View>
  );
};

export default youtube-playlist;

